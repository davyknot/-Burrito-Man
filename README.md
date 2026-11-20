# 🌯 Burrito Man

A top-down, classic-2D delivery game set on the **real streets of Manhattan**.
You run a tiny burrito shop on the Lower East Side. Customers order dozens of burritos at a time and you have to get them there on time: on foot, or in a car you "borrow".
Blocked by a crowd? Throw burritos to scatter them. But every burrito you throw is a burrito you don't get paid for.

**Play:** open `index.html` in a browser. It's one self-contained file (about 3.6 MB) with the whole island inside.

## How to play

| Key | Action |
|---|---|
| WASD / arrows | walk · drive |
| E / Enter | steal the nearest car (the driver gets out, unhappy) / get out |
| Mouse click / F | throw a burrito (aim with the mouse; it snaps to people near the cursor) |
| Space | handbrake (drift) |
| H | horn: pedestrians step aside (crowds don't care) |
| M / Tab | map of all of Manhattan |
| R | car radio · N mute · P pause |

On a phone you get a joystick plus THROW / CAR / BRAKE / MAP buttons.

* **Orders:** walk or drive into the glowing circle at the shop. You get N burritos, a real Manhattan address, and a timer. Follow the dotted GPS line. Faster delivery means a bigger tip; late delivery pays half.
* **Burritos as ammo:** a hit makes people scream and run, and everyone nearby panics. Each throw costs you the sale price.
* **Hero moments:** purse snatchers show up now and then. Hit the thief with a burrito, return the purse, and get cash plus Respect.
* **Heat 🌶 (wanted level):** running people over, driving on sidewalks, stealing cars (especially police cars), ramming cops, or throwing burritos *at* cops raises your heat. Cops chase you, and burritos don't stop them. If they box you in, or catch you on foot, you're **BUSTED**: you pay a fine and the cops eat the order. Lie low and the heat cools down.
* **Street chaos:** traffic lights cause real jams, broken-down trucks block lanes, and breakdance or flash-mob crowds block your route.
* **Shop levels:** Cart → Stand → Shop → Restaurant → BURRITO PALACE. Bigger orders, better prices, and the Burrito Van (it turns gold at the top level). Progress is saved in your browser.

## Host it on GitHub Pages

1. Create a repo and push this folder. The repo only needs `index.html`; the rest is source.
2. Repo → Settings → Pages → *Deploy from a branch* → `main` / root.
3. Play at `https://<you>.github.io/<repo>/`.

## How it's built

```
tools/extract_pbf.py   OSM extract (.osm.pbf) → data/raw/*.json      (Manhattan cut out by its borough boundary)
tools/fetch_osm.py     alternative: same data via the Overpass API     (often overloaded)
tools/build_map.py     raw OSM → data/map.bin.gz  (the game world)
tools/build.py         src/* + map → index.html   (single file, map embedded as base64)
src/
  world.js     map decoding, 2 m surface grid, building collision, road graph + A*, traffic lights,
               cached ground chunks, buildings with GTA-2-style perspective
  entities.js  car physics (bicycle model, drift, rigid-body car-to-car impulses), people, burritos, particles
  traffic.js   traffic AI (lanes, one-ways, lights, honking), pedestrian AI, police pursuit, spawning
  game.js      orders, money, heat/police, events, camera, HUD, minimap
  audio.js     all sound synthesized live with WebAudio: jingle, screams, engine, siren, radio
```

**Map pipeline.** The map comes from OpenStreetMap's vector data, not Google Maps. Google's terms don't allow extracting map data, and detecting buildings in images is unreliable. OSM already has every building outline (45,000, with heights where known), every road with lanes and one-way tags, the parks, and about 6,000 named addresses used as delivery targets. The map is rotated 29° so the avenues run straight up the screen.

**What makes it a world, not a picture:** buildings are real collision polygons, and water blocks you. Cars are physics bodies; traffic follows the actual road graph, drives on the right, respects one-ways and stops at lights. People walk the sidewalks and cross streets. Only the area around the camera is simulated, so the whole island stays fast.

Rebuild the map (needs Python with numpy, scipy, shapely, Pillow, osmium):

```sh
python3 -m venv --system-site-packages .venv && .venv/bin/pip install osmium
curl -o data/raw/NewYork.osm.pbf https://download.bbbike.org/osm/bbbike/NewYork/NewYork.osm.pbf
.venv/bin/python tools/extract_pbf.py data/raw/NewYork.osm.pbf
.venv/bin/python tools/build_map.py
python3 tools/build.py
```

After changing code in `src/`, run `python3 tools/build.py` again.

## Ideas for next steps

* Multi-stop orders, rival delivery bikes, rush hour, a day/night cycle
* Bridges and tunnels to Brooklyn and New Jersey (the map currently stops at the water)
* Hand-drawn sprites instead of the procedural cars and people
* Upgrades: insulated bag (more burritos), scooter, burrito cannon

Map data © OpenStreetMap contributors, available under the ODbL.
