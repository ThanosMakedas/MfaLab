# Övning 5.3 — MFA med TOTP, kontolåsning och recovery codes

Athanasios Makedas

## 1. Varför TOTP och inte SMS i den här applikationen?

Jag valde TOTP därför att den delade hemligheten byts en enda gång, vid
registreringen av autentiseringsappen, och därefter räknar telefonen och
servern fram samma sexsiffriga kod var för sig utan att något skickas över
nätet. SMS måste i stället transporteras genom mobiloperatörens nät i varje
inloggning, vilket öppnar för SIM-kapning där angriparen porterar numret till
sitt eget kort, för avlyssning i signaleringsnätet och för det enklaste av
allt: en kod som syns på en låst skärm. Att appen redan förutsätter en
smartphone gör TOTP till det billigare valet också praktiskt, eftersom det
varken kostar per meddelande eller gör inloggningen beroende av en operatör.
Jag vill ändå vara tydlig med att TOTP inte skyddar mot en phishing-proxy som
står mellan användaren och sajten i realtid, för då vidarebefordras den
sexsiffriga koden inom sitt trettiosekundersfönster precis som en SMS-kod
skulle ha gjort. Slutsatsen är att TOTP höjer ribban tydligt mot de vanligaste
angreppen, men att SMS ändå är klart bättre än ingen andra faktor alls, och
att steget efter TOTP är passkeys som binder inloggningen till domänen.

## 2. Hur landade jag i fem försök och femton minuter?

Fem försök ger utrymme för de felslag en riktig användare gör med fel
tangentbordslayout eller Caps Lock på, samtidigt som det är alldeles för få
för att gissa ett lösenord. Femton minuters låsning sänker gissningstakten
till tjugo lösenord i timmen, vilket gör automatiserad brute force meningslös
utan att en användare som verkligen har glömt sitt lösenord ska behöva ringa
supporten. Avvägningen är att låsningen samtidigt är ett vapen: den som känner
till en e-postadress kan medvetet mata in fem felaktiga lösenord och hålla
kontot stängt, och den attacken blir billigare ju hårdare låsningen är. Därför
valde jag en låsning som släpper av sig själv i stället för en som kräver att
en administratör låser upp, eftersom det senare hade gjort utestängningen
permanent för angriparens räkning. En riktig produktionsapplikation bör
dessutom komplettera kontolåsningen med begränsning per IP-adress, så att
försvaret träffar angriparen snarare än det konto som angrips.

## 3. Observation: recovery codes lagras inte hashade

Uppgiften säger att koderna ska lagras hashade, och repots README påstår att
Identity gör det. Jag kontrollerade i stället i databasen, och det stämmer
inte i den här versionen. Fältet `RecoveryCodes` i `AspNetUserTokens`
innehåller de tio koderna i klartext, sammanfogade med semikolon:

```
antal delar        : 10
dellängder         : 11          (formen xxxxx-xxxxx)
teckenuppsättning  : -23456789BCDFGHJKMNPQRTVWXY
```

Teckenuppsättningen saknar systematiskt A, E, I, L, O, S, U, Z, 0 och 1,
alltså precis de tecken som är lätta att förväxla när en människa läser av
en kod från ett papper. Ett hashvärde har ingen sådan alfabetsbegränsning och
skulle dessutom vara betydligt längre än elva tecken. Koderna ligger med
andra ord i klartext.

Det här är värt att skilja från autentiseringsnyckeln. `AuthenticatorKey`
ligger också i klartext, men **måste** göra det, eftersom servern behöver
själva hemligheten för att räkna fram samma sexsiffriga kod som telefonen.
Recovery codes ska däremot bara jämföras, precis som lösenord, och skulle
därför kunna lagras hashade. Att de inte gör det betyder att den som kommer
åt databasen får tio färdiga förbigångar av tvåfaktorn.

## Verifieringar

| Vad | Hur det verifierades |
|-----|----------------------|
| TOTP påslaget | `TwoFactorEnabled = True` i `AspNetUsers`, samt att inloggning kräver kod (`docs/totp-2fa-prompt.png`) |
| Låsning vid femte försöket | `AccessFailedCount` räknade 1–4, femte försöket satte `LockoutEnd` 15 minuter fram och nollställde räknaren (`docs/lockout-efter-fem-forsok.png`) |
| Låsningen gäller alla | Även korrekt lösenord nekades under låsningen, eftersom `PasswordSignInAsync` kontrollerar `LockoutEnd` före lösenordet |
| Låsningen släpper | Inloggning med korrekt lösenord fungerade igen efter att `LockoutEnd` passerat, utan någon administratörsåtgärd |
| Recovery code fungerar en gång | Första användningen loggade in. Antalet koder i `AspNetUserTokens` gick från 10 till 9, alltså tas koden bort ur listan i stället för att märkas som använd. Samma kod en andra gång gav "Invalid recovery code entered" (`docs/recovery-code-forbrukad.png`) |
| Misslyckad recovery code låser inte | `AccessFailedCount` stod kvar på 0 efter det felaktiga försöket. Bara felaktigt lösenord och felaktig TOTP-kod räknas mot låsningen |
