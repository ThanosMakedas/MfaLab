# Skärmbilder, övning 5.3

Bevis för att TOTP är aktiverat och verifierat med Microsoft Authenticator.

| Fil | Visar |
|-----|-------|
| `totp-2fa-prompt.png` | Efter korrekt lösenord skickas användaren till `/Account/LoginWith2fa` och måste ange den sexsiffriga koden. Beviset på att `SetTwoFactorEnabledAsync` har slagit på tvåfaktor. |
| `totp-inloggad.png` | Lyckad inloggning efter att koden ur autentiseringsappen har verifierats. |

Skärmbilderna innehåller medvetet varken QR-koden, den delade hemligheten
eller några recovery codes.
