# Skärmbilder, övning 5.3

Bevis för TOTP och kontolåsning.

| Fil | Visar |
|-----|-------|
| `totp-2fa-prompt.png` | Efter korrekt lösenord skickas användaren till `/Account/LoginWith2fa` och måste ange den sexsiffriga koden. Beviset på att `SetTwoFactorEnabledAsync` har slagit på tvåfaktor. |
| `totp-inloggad.png` | Lyckad inloggning efter att koden ur autentiseringsappen har verifierats. |
| `lockout-efter-fem-forsok.png` | Det femte felaktiga lösenordet ger `result.IsLockedOut` och skickar användaren till `/Account/Lockout`. Därefter nekas även det korrekta lösenordet tills låsningen släpper. |

Skärmbilderna innehåller medvetet varken QR-koden, den delade hemligheten
eller några recovery codes.
