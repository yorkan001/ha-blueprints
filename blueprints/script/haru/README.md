# Siguran impuls za vrata – Home Assistant blueprint (skripta)

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fyorkan001%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fscript%2Fharu%2Fsiguran_impuls_vrata.yaml)

Autor: Haru Home Assistant (netvis.hr) · Minimalna verzija HA: 2024.10.0

## Čemu služi
Vrata dvorišta ili garaže koja nemaju fotoćelije upravljaju se jednim impulsom (otvori – stani – zatvori).
Ako se drugi impuls pošalje dok se vrata kreću, vrata stanu ili krenu natrag. Ovaj blueprint pravi
**jednu skriptu** koja šalje **točno jedan impuls**, i to samo kad je sigurno:

1. relej je dostupan i u stanju mirovanja (isključen, ili uključen za način `turn_off`);
2. relej se nije mijenjao zadano vrijeme (zadano 60 s) – inače se vrata vjerojatno još kreću;
3. senzor vrata je dostupan;
4. senzor je u dozvoljenom stanju (zadano: *zatvorena*) barem zadano vrijeme (zadano 5 s).

Ako bilo što nije u redu, impuls se **ne šalje**, pokreću se (opcionalne) akcije „Kad je odbijeno”, a
dostupna je varijabla `razlog`, npr. `relej nedostupan`, `vrata se kreću`, `senzor nedostupan`,
`vrata nisu zatvorena`, `vrata nisu otvorena`. Skripta **nikad** ne šalje drugi, „korektivni” impuls.
Nakon impulsa skripta još čeka (vrijeme mirovanja releja, najviše 20 s) i za to vrijeme tiho
ignorira ponovljene pritiske.

## Uvoz (import)
**Način A – preko poveznice:** Postavke → Automatizacije i scene → Blueprinti → *Uvezi blueprint* →
zalijepite URL do datoteke `siguran_impuls_vrata.yaml` (npr. GitHub/Gist „raw” poveznica) → *Pregled* → *Uvezi*.

**Način B – kopiranjem datoteke:** kopirajte `siguran_impuls_vrata.yaml` u
`/config/blueprints/script/haru/` (mapu napravite ako ne postoji), zatim u
Postavke → Automatizacije i scene → Blueprinti kliknite ⋮ → *Ponovno učitaj blueprinte*
(ili ponovno pokrenite HA).

## Izrada skripte
1. Postavke → Automatizacije i scene → **Blueprinti** → „Siguran impuls za vrata” → *Izradi skriptu*.
2. Odaberite:
   - **Relej za impuls** – prekidač (switch) koji daje impuls vratima;
   - **Način impulsa**:
     - `turn_on` – relej inače isključen; impuls je uključenje. Na samom uređaju uključite
       automatsko isključenje (npr. Shelly *Auto off* 1 s) – **preporučeno**;
     - `turn_off` – relej inače uključen (npr. Sonoff MINI-ZBD); impuls je isključenje;
     - `turn_on_then_off` – skripta uključi pa nakon *Trajanja impulsa* isključi relej;
   - **Trajanje impulsa** (0,2–5 s, samo za `turn_on_then_off`);
   - **Kontaktni senzor vrata** i koje stanje znači **otvoreno** (obično `on`);
   - **Impuls dozvoljen kad su vrata**: `zatvorena` (samo otvaranje), `otvorena` (samo zatvaranje) ili `oboje`;
   - **Senzor stabilan barem** (s) i **Relej miran barem** (s);
   - opcionalno **Kad je impuls poslan** (npr. snimka kamere) i **Kad je odbijeno** (npr. obavijest).
3. Spremite skriptu, npr. kao `script.siguran_impuls_vrata_dvoriste`.

Za otvaranje i zatvaranje istih vrata s različitim pravilima možete napraviti dvije skripte
iz istog blueprinta (jednu s `zatvorena`, drugu s `otvorena`).

## Pozivanje
Iz automatizacije, gumba na nadzornoj ploči ili glasovnog asistenta:

```yaml
actions:
  - action: script.siguran_impuls_vrata_dvoriste
```

Primjer akcije „Kad je odbijeno” (obavijest na mobitel):

```yaml
- action: notify.mobile_app_moj_mobitel
  data:
    title: Vrata nisu pokrenuta
    message: "Razlog: {{ razlog }}"
```

## Napomene
- Nakon ponovnog pokretanja HA vrijeme „zadnje promjene” releja i senzora počinje ispočetka, pa
  skripta prvih *Relej miran barem* sekundi odbija impuls (namjerno, sigurnija strana).
- Skripta ne zna je li impuls stvarno pokrenuo vrata – to pokazuje kontaktni senzor.
- Blueprint ne zamjenjuje fotoćelije ni ostalu fizičku zaštitu; vrata pokrećite samo kad ih vidite
  ili kad ste sigurni da u prolazu nema nikoga.
