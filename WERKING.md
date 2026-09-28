# Werking Vectorworks-materiaalbibliotheek

## Wat er op een werkplek komt

De basisoplossing is een Vectorworks 2026 Python-plug-in. Intune plaatst deze bestanden eenmalig in de
persoonlijke Vectorworks-plug-inmap:

- `INTOS_Library_Sync.py`
- `desired_state.py`
- `library_release.py`
- `config.json`

De plug-in gebruikt uitgaand HTTPS naar deze openbare GitHub-repository. Er hoeft geen browser open te staan,
er is geen Chrome-extensie nodig en de gebruiker hoeft geen browsertoestemming te geven. De grote
`INTOS Texturen 2026.vwx` staat niet in Git. Vectorworks bouwt dit bestand op iedere werkplek zelf.

Voor een automatische controle om de vijf minuten is naast de plug-in een kleine Windows-taak nodig. Deze taak
leest alleen `latest.json` en toont een Windows-melding. De taak wijzigt geen Vectorworks-bestanden. De echte
update draait altijd binnen Vectorworks, omdat alleen Vectorworks veilig VWX-resources kan maken en opslaan.

## Gemaakte scripts

| Script                           | Functie                                                                                                                                                                                                                                                                                                |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `INTOS_Library_Sync.py`          | VW26-menuopdracht. Leest een release, controleert hashes en de actieve bibliotheek, maakt of vervangt textures, verwijdert alleen eerder beheerde textures die zijn vervallen, slaat de beheerde VWX op en installeert daarna beide interiorcad-bestanden. Een gewone projecttekening wordt geweigerd. |
| `desired_state.py`               | Voegt de volledige centrale gewenste toestand veilig samen met de lokale bestanden. Alleen eerder geregistreerde INTOS-regels en -textures mogen worden gewijzigd of verwijderd; onbekende lokale inhoud blijft staan en conflicten blokkeren de update.                                               |
| `library_release.py`             | Gemeenschappelijke releasecode. Valideert het manifest, weigert onveilige paden en hashes, downloadt of leest bestanden en zet ze eerst in een tijdelijke map.                                                                                                                                         |
| `build_public_release.py`        | Bouwt een onveranderlijke GitHub-release. Controleert de headers van `Boards.txt` en `EdgeBandings.txt`, texturematen en SHA-256-hashes. Publicatie stopt als een database naar een ontbrekende texture verwijst.                                                                                      |
| `prepare_pilot_release.py`       | Maakt een lokale pilotrelease voor een afgebakende VW26-test. Niet bedoeld als productiepublicatie.                                                                                                                                                                                                    |
| `resolve_autoimport.py`          | Vindt de echte, gelokaliseerde interiorcad Autoimport-map uit de Vectorworks-instellingen en valideert de twee exportbestanden.                                                                                                                                                                        |
| `local_connector.py`             | Eerdere localhost-proef voor communicatie vanuit de webapp. Deze is niet nodig voor de gekozen VW26-oplossing, maar blijft beschikbaar als diagnose- en terugvalroute.                                                                                                                                 |
| `export_current_document.py`     | Leest het geopende bron-VWX en exporteert textureafbeeldingen, namen, UUID's, koppelingen en hashes naar een controleerbaar pakket.                                                                                                                                                                    |
| `run_export_current_document.px` | Dunne VectorScript-wrapper voor omgevingen die de Python-export niet rechtstreeks starten.                                                                                                                                                                                                             |
| `read_bridge.py`                 | Tijdelijke read-only diagnosebridge voor een geopende Vectorworks-sessie. Kan alleen document-, materiaal- en texturegegevens lezen.                                                                                                                                                                   |
| `probe_read_bridge.py`           | Testclient voor de read-only diagnosebridge.                                                                                                                                                                                                                                                           |
| `test_*.py`                      | Tests voor manifesten, hashes, ongeldige paden en maten, ontbrekende textures, herhaalbaarheid en bescherming van projecttekeningen.                                                                                                                                                                   |

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

1. Een medewerker vraagt materiaal of kantenband aan in Materiaalbeheer. Volgens de bestaande aanvraagworkflow staat een ingediende nieuwe materiaalregel direct in de actieve hoofdlijst.
2. Iedere wijziging aan materiaal, decor, kantenband of primaire afbeelding verhoogt de Vectorworks-revisie en zet één samengevoegde publicatiejob klaar.
3. De serverfunctie leest de volledige actuele actieve hoofdlijst en maakt nieuwe `Boards.txt` en `EdgeBandings.txt`.
4. De serverfunctie voegt de bijbehorende textureafbeeldingen en expliciete fysieke texturematen toe en controleert hashes en volledigheid.
5. De server publiceert de onveranderlijke revisiemap en `latest.json` met de Git Data API in één niet-geforceerde Git-commit.
6. Afwijzen, intrekken of deactiveren maakt opnieuw een volledige release. De vervallen regel of texture ontbreekt daarin en wordt lokaal alleen verwijderd wanneer hij eerder als INTOS-beheerd is geregistreerd.
7. De server bundelt wijzigingen gedurende vijf minuten. Iedere nieuwe wijziging herstart dat venster. Na
   25 minuten stopt het uitstellen, zodat de scheduler de publicatie uiterlijk dertig minuten na de eerste
   wijziging start.
8. De werkplek ziet uiterlijk binnen vijf minuten dat een gepubliceerde revisie beschikbaar is.
9. De gebruiker krijgt de melding om werk op te slaan en de beheerde bibliotheekupdate uit te voeren.
10. VW26 controleert alle hashes, bouwt of actualiseert `INTOS Texturen 2026.vwx` en controleert texturemaat en naam.
11. Pas na een volledige VWX-controle vervangt de plug-in samen `Boards.txt` en `EdgeBandings.txt`.
12. De lokale status bewaart revisie, bestandshashes en texturevingerafdrukken. De volgende controle meldt exact wat ontbreekt of verouderd is.

## Veiligheidsregels

- De plug-in schrijft alleen naar het vaste beheerde bibliotheekdocument en nooit naar een projecttekening.
- Iedere download wordt vóór installatie met SHA-256 gecontroleerd.
- Een gedeeltelijke release zonder alle gebruikte textures wordt geweigerd.
- Databasebestanden worden pas geplaatst nadat de VWX-resourcecontrole is geslaagd.
- Twee uitvoeringen van dezelfde revisie leveren dezelfde status op en veroorzaken geen tweede wijziging.
- Eén niet-geforceerde Git-refupdate voorkomt dat twee gelijktijdige publicaties elkaar overschrijven.
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

Voor de live stagingketen moeten migratie 035 en de `vectorworks-release`-serverfunctie nog op de
self-hosted Supabase-stack worden geïnstalleerd. De server krijgt een repo-scoped GitHub-sleutel en een
afzonderlijk schedulergeheim. Voor volledig automatische VWX-wijzigingen terwijl Vectorworks open staat is
daarnaast een beperkte native Vectorworks-bridge nodig; de huidige Python-opdracht start handmatig vanuit
Vectorworks.
