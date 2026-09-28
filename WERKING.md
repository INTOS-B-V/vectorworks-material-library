# Werking Vectorworks-materiaalbibliotheek

## Wat er op een werkplek komt

De basisoplossing is een Vectorworks 2026 Python-plug-in. Intune plaatst deze bestanden eenmalig in de
persoonlijke Vectorworks-plug-inmap:

- `INTOS_Library_Sync.py`
- `library_release.py`
- `config.json`

De plug-in gebruikt uitgaand HTTPS naar deze openbare GitHub-repository. Er hoeft geen browser open te staan,
er is geen Chrome-extensie nodig en de gebruiker hoeft geen browsertoestemming te geven. De grote
`INTOS Texturen 2026.vwx` staat niet in Git. Vectorworks bouwt dit bestand op iedere werkplek zelf.

Voor een automatische controle om de vijf minuten is naast de plug-in een kleine Windows-taak nodig. Deze taak
leest alleen `latest.json` en toont een Windows-melding. De taak wijzigt geen Vectorworks-bestanden. De echte
update draait altijd binnen Vectorworks, omdat alleen Vectorworks veilig VWX-resources kan maken en opslaan.

## Gemaakte scripts

| Script | Functie |
| --- | --- |
| `INTOS_Library_Sync.py` | VW26-menuopdracht. Leest een release, controleert hashes en de actieve bibliotheek, maakt of vervangt textures, verwijdert alleen eerder beheerde textures die zijn vervallen, slaat de beheerde VWX op en installeert daarna beide interiorcad-bestanden. Een gewone projecttekening wordt geweigerd. |
| `library_release.py` | Gemeenschappelijke releasecode. Valideert het manifest, weigert onveilige paden en hashes, downloadt of leest bestanden en zet ze eerst in een tijdelijke map. |
| `build_public_release.py` | Bouwt een onveranderlijke GitHub-release. Controleert de headers van `Boards.txt` en `EdgeBandings.txt`, texturematen en SHA-256-hashes. Publicatie stopt als een database naar een ontbrekende texture verwijst. |
| `prepare_pilot_release.py` | Maakt een lokale pilotrelease voor een afgebakende VW26-test. Niet bedoeld als productiepublicatie. |
| `resolve_autoimport.py` | Vindt de echte, gelokaliseerde interiorcad Autoimport-map uit de Vectorworks-instellingen en valideert de twee exportbestanden. |
| `local_connector.py` | Eerdere localhost-proef voor communicatie vanuit de webapp. Deze is niet nodig voor de gekozen VW26-oplossing, maar blijft beschikbaar als diagnose- en terugvalroute. |
| `export_current_document.py` | Leest het geopende bron-VWX en exporteert textureafbeeldingen, namen, UUID's, koppelingen en hashes naar een controleerbaar pakket. |
| `run_export_current_document.px` | Dunne VectorScript-wrapper voor omgevingen die de Python-export niet rechtstreeks starten. |
| `read_bridge.py` | Tijdelijke read-only diagnosebridge voor een geopende Vectorworks-sessie. Kan alleen document-, materiaal- en texturegegevens lezen. |
| `probe_read_bridge.py` | Testclient voor de read-only diagnosebridge. |
| `test_*.py` | Tests voor manifesten, hashes, ongeldige paden en maten, ontbrekende textures, herhaalbaarheid en bescherming van projecttekeningen. |

## Release-inhoud

Een actieve release bestaat uit:

- `Boards.txt`, de goedgekeurde plaatmaterialen voor interiorcad;
- `EdgeBandings.txt`, de goedgekeurde kantenbanden voor interiorcad;
- textureafbeeldingen met een exacte resourcenaam en fysieke breedte in millimeters;
- `release.json`, met revisie, bestanden, maten en SHA-256-hashes;
- `latest.json`, de kleine verwijzing naar de actieve onveranderlijke revisie.

De mapindeling is `releases/<kanaal>/<revisie>/`. Staging en productie krijgen ieder hun eigen kanaal. Een oude
revisie blijft bestaan en kan daardoor direct weer als `latest.json` worden aangewezen.

## Proces van aanvraag tot Vectorworks

1. Een medewerker vraagt materiaal of kantenband aan in Materiaalbeheer.
2. Een beheerder controleert en keurt de aanvraag goed.
3. Materiaalbeheer maakt nieuwe `Boards.txt` en `EdgeBandings.txt` uit de actuele goedgekeurde hoofdlijst.
4. Het releaseproces voegt de bijbehorende textureafbeeldingen en fysieke texturematen toe.
5. `build_public_release.py` controleert of iedere genoemde texture aanwezig is en bouwt de release.
6. De release wordt als één Git-commit gepubliceerd. Pas daarna wijst `latest.json` naar de nieuwe revisie.
7. De werkplek ziet uiterlijk binnen vijf minuten dat een nieuwe revisie beschikbaar is.
8. De gebruiker krijgt de melding om werk op te slaan en de beheerde bibliotheekupdate uit te voeren.
9. VW26 controleert alle hashes, bouwt of actualiseert `INTOS Texturen 2026.vwx` en controleert texturemaat en naam.
10. Pas na een volledige VWX-controle vervangt de plug-in samen `Boards.txt` en `EdgeBandings.txt`.
11. De lokale status bewaart revisie, bestandshashes en texturevingerafdrukken. De volgende controle meldt exact wat ontbreekt of verouderd is.

## Veiligheidsregels

- De plug-in schrijft alleen naar het vaste beheerde bibliotheekdocument en nooit naar een projecttekening.
- Iedere download wordt vóór installatie met SHA-256 gecontroleerd.
- Een gedeeltelijke release zonder alle gebruikte textures wordt geweigerd.
- Databasebestanden worden pas geplaatst nadat de VWX-resourcecontrole is geslaagd.
- Twee uitvoeringen van dezelfde revisie leveren dezelfde status op en veroorzaken geen tweede wijziging.
- Oude Git-revisies blijven beschikbaar voor rollback.

## Huidige pilotstatus

De lokale VW26-proef heeft één texture gemaakt, de beheerde bibliotheek opgeslagen, beide databasebestanden
geïnstalleerd en een tweede uitvoering zonder wijziging afgerond. Ook is getest dat uitvoering vanuit een gewone
projecttekening wordt gestopt.

Op staging is aanvraag `#33`, `HPL 0.82mm-Abet 602 Sei eik`, ingediend en goedgekeurd. De nieuwe Boards-export
bevat 816 regels en de aangevraagde regel. De EdgeBandings-export bevat 365 bestaande kantenbandregels. Er is
geen nieuwe kantenband aangevraagd, waardoor dat aantal niet wijzigde.

Er staat nog geen actieve release in deze openbare repository. De database verwijst momenteel naar meer
textures dan in het pilotpakket zitten. Productiepublicatie blijft geblokkeerd totdat alle gebruikte textures
zijn geëxporteerd, gecontroleerd en opgenomen.

## Stappen voor INTOS-brede uitrol

1. VW26-bronbibliotheek volledig exporteren en alle databaseverwijzingen aan een texture koppelen.
2. Een complete stagingrelease bouwen en op enkele testlaptops installeren.
3. Update, herstart, ontbrekende texture, beschadigde download en rollback testen.
4. De Intune-installatie maken voor plug-in, configuratie en de optionele vijfminutencontrole.
5. Eerst een beperkte gebruikersgroep activeren en statusmeldingen verzamelen.
6. Na akkoord dezelfde gevalideerde revisie naar het productiekanaal promoveren.
7. Daarna gefaseerd uitrollen naar alle VW26-gebruikers.

Voor productie moeten nog twee punten worden toegevoegd: een ondertekend of vastgepind manifest en één lock
tegen twee gelijktijdige updateruns. De huidige pilot is daarom geschikt voor lokale en stagingtests, maar nog
niet voor onbeheerde bedrijfsbrede installatie.
