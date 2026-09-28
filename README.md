# INTOS Vectorworks material library

Openbare, versieerbare releasebestanden voor de INTOS Vectorworks 2026-materiaalbibliotheek.

De repository bevat alleen materiaaldefinities, interiorcad-importbestanden en textureafbeeldingen.
Projecttekeningen, klantgegevens, aanvragen, gebruikersgegevens en toegangsgegevens horen hier niet in.

Vectorworks leest per kanaal `releases/<kanaal>/latest.json`. Iedere publicatie verwijst naar bestanden
in een eigen onveranderlijke revisiemap. De client controleert alle SHA-256-hashes voordat hij iets installeert.
Het grote `.vwx`-bestand staat niet in Git. Vectorworks 2026 bouwt dit lokaal uit de gepubliceerde afbeeldingen.

De staging-URL wordt na publicatie:

`https://raw.githubusercontent.com/INTOS-B-V/vectorworks-material-library/main/releases/staging/latest.json`

Er staat pas een actieve `latest.json` in deze repository wanneer iedere texture waarnaar Boards.txt of
EdgeBandings.txt verwijst in dezelfde release staat. Een gedeeltelijke release kan daardoor nooit per ongeluk
de bibliotheek van een collega vervangen. De lokale VW26-pilot gebruikt tot die tijd een lokale testrelease.

Publiceren gebeurt met `tools/vectorworks/build_public_release.py` uit de applicatierepository. Commit eerst
de nieuwe revisiemap en wijzig `latest.json` in dezelfde commit. Pas daarna mag een client de revisie gebruiken.

Zie [WERKING.md](WERKING.md) voor de gemaakte scripts, installatie op werkplekken, het releaseproces en de
stappen van de pilot naar een INTOS-brede uitrol.
