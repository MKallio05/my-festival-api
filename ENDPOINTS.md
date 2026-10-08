# My Festival API — endpoint-dokumentti / endpoint document

> FI: Täytä tämä pohja ja palauta se Moodleen (Markdown tai PDF, 1–2 sivua). Korvaa kaikki `<…>`-kohdat.
> Esimerkkivastaukset kopioit oman julkisen rajapintasi oikeista vastauksista (selain tai `curl.exe`/`curl`).
>
> EN: Fill in this template and submit it to Moodle (Markdown or PDF, 1–2 pages). Replace every `<…>`.
> Copy the example responses from your own public API's real responses (browser or `curl.exe`/`curl`).

**Tekijä / Author:** <Matias Kallio>
**Julkinen osoite / Public URL:** `https://festival-api-festival-matikall.2.rahtiapp.fi`
**GitHub:** `https://github.com/MKallio05/my-festival-api`
**Tarkistimen tulos / Checker result:** 

[←[92mPASS←[0m] GET /api/artists returns 200
[←[92mPASS←[0m]   Content-Type is application/json
[←[92mPASS←[0m]   response is a non-empty JSON list
[←[92mPASS←[0m]   items have artist_id and name
[←[92mPASS←[0m] GET /api/days returns day_id + date
[←[92mPASS←[0m] GET /api/stages returns stage_id + name
[←[92mPASS←[0m] GET /api/artists/10/events returns a JSON list
[←[92mPASS←[0m] GET unknown artist returns 404
[←[92mPASS←[0m] GET /api/timetable returns a JSON list
[←[92mPASS←[0m] GET /api/timetable?day=2025-07-05 returns only that day
[←[92mPASS←[0m]   rows are in time order (starts)
[←[92mPASS←[0m] GET timetable with bad date returns 400
[←[92mPASS←[0m] POST with missing fields returns 400
[←[92mPASS←[0m] POST with bad foreign key returns 400
[←[92mPASS←[0m] POST with bad start_dt format returns 400
[←[92mPASS←[0m] POST with valid body returns 201 + event_id
[←[92mPASS←[0m]   new event is returned by GET /api/artists/{id}/events
[←[92mPASS←[0m] DELETE /api/events/123 returns 204
[←[92mPASS←[0m]   deleted event is gone
[←[92mPASS←[0m] DELETE same event again returns 404

20/20 checks passed` (liitä koko tuloste palautukseen / attach the full output)

---

## 1. Endpointit / Endpoints

| Metodi / Method | Polku / Path | Onnistuu / Success | Virheet / Errors |
|---|---|---|---|
| GET | `/api/artists` | 200 | – |
| GET | `/api/days` | 200 | – |
| GET | `/api/stages` | 200 | – |
| GET | `/api/artists/{id}/events` | 200 | 404 |
| GET | `/api/timetable?day=YYYY-MM-DD` | 200 | 400 |
| POST | `/api/events` | 201 | 400 |
| DELETE | `/api/events/{id}` | 204 | 404 |

## 2. Esimerkit / Examples

Kirjoita jokaisesta endpointista esimerkkikutsu ja (lyhennetty) vastaus. Malli:
For every endpoint, write an example call and a (shortened) response. Example:

### GET /api/artists

```bash
curl https://festival-api-festival-matikall.2.rahtiapp.fi/api/artists
```
```json
[{"artist_id":10,"country":"UK","name":"Adele"},{"artist_id":17,"country":"FIN","name":"Alma"}]...
```

### GET /api/artists/{id}/events
```bash
curl https://festival-api-festival-matikall.2.rahtiapp.fi/api/artists/10/events
```
```json
[{"day":"2025-07-05","event_id":1,"stage":"Main A","starts":"14:15:00"},{"day":"2025-07-05","event_id":27,"stage":"Club","starts":"17:15:00"}]....
```


Tuntematon id / unknown id → `<tilakoodi ja runko / status code and body>`
```bash
curl https://festival-api-festival-matikall.2.rahtiapp.fi/api/artists/10000/events
```
```json
{"error":"artist 10000 not found"}
```




### GET /api/timetable?day=YYYY-MM-DD
```bash
curl https://festival-api-festival-matikall.2.rahtiapp.fi/api/timetable?day=2025-07-06
```
```json
[{"artist":"Imagine Dragons","date":"2025-07-06","ends":"14:40:00","stage":"Club","starts":"14:15:00"},{"artist":"Adele","date":"2025-07-06","ends":"15:00:00","stage":"Main A","starts":"14:15:00"}]...
```


Virheellinen päivämäärä / invalid date → `<tilakoodi ja runko / status code and body>`
```bash
curl https://festival-api-festival-matikall.2.rahtiapp.fi/api/timetable?day=2025-4323
```
```json
{"error":"invalid date, use YYYY-MM-DD"}
```



### POST /api/events
```bash
curl -i -X POST https://festival-api-festival-matikall.2.rahtiapp.fi/api/events \
     -H "Content-Type: application/json" \
     -d '{"artist_id": 1, "stage_id": 1, "day_id": 1, "start_dt": "2026-07-10 20:00:00"}'
```
```json
<artist_id         : 1
day_id            : 1
duration_min      : 45
end_dt            : 2026-07-10 20:45:00
event_id          : 127
event_type_id     : 1
previous_event_id :
stage_id          : 1
start_dt          : 2026-07-10 20:00:00>
```
Puuttuva kenttä / missing field → `{"error":"missing fields: stage_id, day_id"}` · Olematon artist_id / unknown artist_id → `{"error":"integrity error: UNIQUE constraint failed: Event.day_id, Event.stage_id, Event.start_dt"}`

### DELETE /api/events/{id}
```bash
curl -X DELETE https://festival-api-festival-matikall.2.rahtiapp.fi/api/events/10
```
Onnistuu / success → `<tyhjä output>` · Tuntematon id / unknown id → `{"error":"event 10 not found"}`

## 3. Julkaisu Rahtiin / Deploying to Rahti (3–6 lausetta / sentences)

<FI: Kuvaa omin sanoin build → deployment → route: Annoin Rahdille GitHub-reponi osoitteen, ja Rahti haki koodin sieltä ja ajoi buildin: se luki Dockerfilen, asensi riippuvuudet ja rakensi konttikuvan, jonka sisään tuli myös SQLite-tietokanta. Deployment käynnisti kuvasta podin, jossa gunicorn ajaa Flask-sovellusta. Service ohjaa liikenteen podiin, ja Route antaa sovellukselle julkisen HTTPS-osoitteen jonka löysin heti kun deployment oli valmis.>
<EN: Describe build → deployment → route in your own words: what Rahti did with your repo and where you found
the public address. Mention one problem you ran into and how you solved it.>

## 4. Tietoturva / Security (2–4 lausetta / sentences)

<FI: SQL-injektion esin käyttämällä parametrisoituja kyselyitä: kaikki käyttäjän syöte (`artist_id`, `day`, `event_id` ja POST-bodyn kentät) annetaan SQL-lauseelle erillisinä `?`-parametreina, eikä sitä liitetä koskaan itse SQL-merkkijonoon. Lisäksi syötteet validoidaan ennen kantaa (esim. `valid_date`, kokonaislukutarkistukset ja `start_dt`:n tiukka muoto).>
<EN: What would this API need before real production use? How did you make sure SQL injection is not possible?>
