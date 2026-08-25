# Skärmbilder, övning 5.3

Bevis för TOTP, kontolåsning och recovery codes.

| Fil | Visar |
|-----|-------|
| `totp-2fa-prompt.png` | Efter korrekt lösenord skickas användaren till `/Account/LoginWith2fa` och måste ange den sexsiffriga koden. Beviset på att `SetTwoFactorEnabledAsync` har slagit på tvåfaktor. |
| `totp-inloggad.png` | Lyckad inloggning efter att koden ur autentiseringsappen har verifierats. |
| `lockout-efter-fem-forsok.png` | Det femte felaktiga lösenordet ger `result.IsLockedOut` och skickar användaren till `/Account/Lockout`. Därefter nekas även det korrekta lösenordet tills låsningen släpper. |
| `recovery-code-forbrukad.png` | Samma recovery code en andra gång ger "Invalid recovery code entered". Koden är förbrukad efter första användningen. |

Skärmbilderna innehåller varken QR-koden eller den delade hemligheten. I
`recovery-code-forbrukad.png` är inmatningsfältet övertäckt: koden är visserligen
redan förbrukad och borttagen ur databasen, men den hör ändå inte hemma i ett repo.
