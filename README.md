# Lumen Rail

A 3D train-ticket booking site. You pick a route, board the train, and walk down the aisle to choose your seat. It's a single `index.html` built with plain HTML, CSS and three.js, with no build step.

All stations, trains, fares and occupancy are fictional and generated in code. Booking is a mock.

## Run it

Serve the folder over HTTP. ES modules and the driver's face texture don't load from `file://`.

```sh
python3 -m http.server 8790
# open http://localhost:8790/
```

You need an internet connection for fonts (Google Fonts) and three.js (jsDelivr).

## Prompt to replicate this site

Paste the prompt below into an AI coding assistant to rebuild the site from scratch.

```text
Build "Lumen Rail": a train-ticket booking website where you choose your seat by
walking through a 3D train. Put everything in a single index.html using plain
HTML, CSS and an ES module script. Don't use a build step or a framework. Load
three.js 0.160.0 from jsDelivr through an import map, and use these addons:
RoundedBoxGeometry, BufferGeometryUtils (mergeGeometries), EffectComposer,
RenderPass, UnrealBloomPass, OutputPass and RoomEnvironment. Fonts come from
Google Fonts: "Big Shoulders Display" for display text, "Geist" for UI text and
"Geist Mono" for numbers and codes.

LOOK AND FEEL
- Night-time train station with a warm reading-lamp mood. The canvas fills the
  whole screen and a dark glass UI floats over it.
- Colours as CSS custom properties: night #0e0c0b, ink #f5f0e7, lamp accent
  #ffd49a, ticket paper #f7f2e8 with paper ink #1d1915, alert #ffb0a0, plus
  translucent glass panels with backdrop blur and hairline borders.
- Logo: a rounded lamp-coloured square with two rails converging toward a lamp
  at the top. Define it once as an SVG <symbol> and reuse it for the favicon,
  the header, the loader, the ticket and a livery decal painted on the train.
- Layout: a top bar with the brand and a 4-step stepper (Journey, Train, Seat,
  Ticket; later steps stay disabled until you reach them) and a close/back
  button. The side sheet sits on the left on desktop and becomes a bottom sheet
  on phones. A seat bar runs along the bottom. Respect safe-area insets and keep
  a 16px minimum gutter.
- A loader shows the logo, a progress bar and the text "Bringing the train
  into the platform…".
- Respect prefers-reduced-motion by shortening tweens and waits.

DATA (all fictional, deterministic)
- Stations: Mumbai CSMT, Pune Junction, New Delhi, Jaipur Junction, Ahmedabad
  Junction, Madgaon (Goa), KSR Bengaluru and Chennai Central, each with a code.
- Trains: "LR 2021 Lumen Express" (19:22, 5 coaches), "LR 1408 Monsoon Line"
  (20:10, 6 coaches) and "LR 0912 Midnight Glide" (22:45, 4 coaches). Each has
  an accent colour, a base duration, base fares for Standard, First and Sleeper,
  and an occupancy rate. Scale duration and fare by a hash of the route, and
  derive arrival time from them. Show prices in rupees (₹, en-IN formatting).
- Coach classes: coach 1 is First. The rear coach is Sleeper (the last two
  coaches when a train has 5 or more). Every other coach is Standard.
- Layouts:
  - Standard: 2+2 seats (A B | C D), 16 rows, with table bays where one row
    faces backwards.
  - First: 2+1 seats (A B | C, where C is a solo window seat), 11 rows, also
    with table bays.
  - Sleeper: Indian-style bays of 8 berths. Each bay has Lower, Middle and
    Upper berths facing each other, plus Side lower and Side upper across the
    aisle. There are 7 bays per coach.
- Taken seats come from a seeded PRNG keyed on train, date and route, so the
  same search always gives the same train. Seats you have already booked show
  as yours.
- You can book up to 6 seats. There is a ₹40 reservation charge per seat.
  Save bookings to localStorage.

THE FLOW
1. Journey: the heading reads "Choose your seat from inside the train" under
   the eyebrow "Night services · Platform 4". The form has From and To
   pickers, a swap button, and a date of journey (default: tomorrow).
   - Native <select> and date inputs can't be themed, so build a custom
     listbox popover for stations and a custom calendar popover for the date.
   - Both popovers need full keyboard support: arrow keys, Home/End,
     Enter/Space and Escape.
   - Validate that From and To differ, then "Find trains".
2. Train: show the route line and date, and a list of trains with departure,
   arrival, duration and fares per class. Choosing a train swaps the 3D train
   on the platform.
   - Show a row of coach chips with class and free-seat counts.
   - Hovering a coach on the 3D train lights its windows and shows a tooltip.
   - Clicking a coach (or its chip) boards it: the camera flies to the door,
     the screen fades, and you appear inside.
3. Seat: a first-person walk through the train.
   - Drag down or up to walk forward or back. Drag sideways to turn a full 360°.
   - Arrow keys or WASD also walk and turn. The mouse wheel walks too.
   - Press V (or use a toggle button) to switch to a 360° look mode where
     dragging looks around (pitch and yaw) instead of walking.
   - Walking has inertia, and the camera keeps to the aisle.
   - Hovering a free seat highlights it. Clicking selects or deselects it.
     Selected seats get a glowing shell and a floating "Reserved" card with the
     seat number above them.
   - The bottom bar shows chips for each selected seat and a CTA that reads
     "Pick a seat" until you have chosen one. A coach menu jumps between
     coaches.
   - Show a hint, "Drag down to walk forward", that fades after a few steps,
     and short toasts for errors such as trying to book more than 6 seats.
4. Ticket: a paper ticket card with a perforation between two sections.
   - It shows the brand, train id and name, from and to (codes, names and
     times), duration, date, coach(es), seat or berth, class, position
     (window, aisle, lower berth and so on) and extras (table, rear-facing).
   - Add one passenger-name input per seat and validate that each name is at
     least 2 characters.
   - Show a fare breakdown (base fare, reservation charge, total) and a
     "Confirm booking · ₹X" button.
   - On confirm, show a "Booked" stamp, the passenger list, a generated SVG
     barcode, and a 10-digit PNR grouped as 3-3-4.
   - Then offer "Book another journey" and "Back to the train". Mark the booked
     seats as taken in the 3D scene.

EXTERIOR 3D SCENE (station at night)
- Tracks, a platform with a canopy and hanging lamps, hanging platform signs,
  a distant city skyline and a starfield.
- A departure board drawn on a canvas texture that lists the trains.
- The train: rounded coach bodies in the train's accent colour with a window
  band, doors, coach number plates, the Lumen Rail logo decal on both sides,
  and underframes, bogies and roof details.
- Streamlined bullet-train noses at both ends, built as a parametric surface
  lofted from the coach profile to a low rounded tip, with a cockpit canopy
  bulge, a wrap-around windscreen, and a bogie under the nose so it sits on
  the track.
- The camera slowly drifts around the train. Dragging or scrolling orbits it,
  and a "Reset view" button returns it to the auto camera.
- Use HDR emissive lamp colours with an UnrealBloomPass so the lights glow, and
  RoomEnvironment for reflections.

INTERIOR 3D SCENE
- One continuous interior for the whole train. The upper walls have real
  window openings, so the station rendered from a matching exterior camera
  shows through the windows.
- An arched ceiling, luggage racks (seated coaches only), aisle runners with
  dots, and partitions between coaches with sliding glass doors and coach signs.
- Seats are InstancedMeshes with an index map so a raycast hit resolves to a
  seat. Sleeper coaches have berths with pillows, bay partitions, side-berth
  end panels, chains for the middle berths, and reading lights.
- Seat number plates come from one texture atlas in one merged mesh. Add
  tables at the table bays.
- Taken seats have simple passengers with varied tints and bags. Taken berths
  have sleeping passengers with a blanket and a head on the pillow. Put luggage
  on the racks.
- Keep draw calls low with instancing and merged geometry. Dispose of the
  interior when you leave it.

DRIVER'S CAB
- The front of the train is a walkable driver's cab with a windscreen wall, a
  console with a sloped desk, two screens drawn on canvas (one with route info
  for the current train), buttons and a power/brake controller.
- The driver sits in a swivel chair facing the windscreen. Wrap a face photo
  (assets/driver-face.png) onto the front of his head, and draw a fallback face
  on a canvas if the image fails to load. Add a little emissive light so the
  face stays readable in the dim cab.
- Clicking the driver makes him swivel round, wave and show a speech bubble
  introducing himself as "Ayush" (label: Driver). Then he turns back.
- Add buttons in the seat view to jump to the cab and back to the seats.

POLISH
- Tween camera moves with easing, and fade to black on board and exit.
- Add hover and drag cursors, focus-visible outlines in the lamp colour, and
  aria labels on the canvas, steppers and pickers. Escape acts as back.
- Detect coarse pointers and adjust the field of view on resize so the aisle
  reads well on phones.
- Generate every texture (signs, board, plates, screens) on a canvas at
  runtime. The only image file is the driver's face.
```
