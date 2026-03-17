# 🎂 Abhiram's Interactive Birthday Card — Final Build Prompt
### Complete specification for the fully built v11 app

---

## PROJECT OVERVIEW

A **single-file, self-contained HTML app** — an interactive digital birthday card for **Abhiram's 2nd birthday**. It is a 24-slide storybook, one slide per month of his life, each with a unique interactive mini-game. The app is fully client-side with no backend, deployable by dropping a single `.html` file onto Netlify.

**Tech stack:** Vanilla HTML + CSS + JavaScript. Web Audio API for all sounds (no external audio files). All assets embedded inline. Single `index.html` file.

---

## DESIGN SYSTEM

### Fonts (Google Fonts)
- **Display/Headings:** `Baloo 2` (weights 400, 600, 700, 800, 900) — round, playful
- **Body:** `Nunito` (weights 400, 600, 700, 800) — soft, readable

### Colour Palette (ChooChoo TV / baby-bright)
```css
:root {
  --sky:   #5BC8F5;   /* Primary blue */
  --sun:   #FFD93D;   /* Yellow accents */
  --berry: #FF6B6B;   /* Red/pink highlights */
  --mint:  #6BCB77;   /* Nature green */
  --lav:   #C77DFF;   /* Lavender magic */
  --peach: #FFAB76;   /* Warm skin tones */
  --white: #FFFDF7;   /* Card backgrounds */
  --navy:  #1A2C5B;   /* Text / borders */
  --cream: #FFF8E7;   /* Warm off-white */
  --gum:   #FF85A1;   /* Bubblegum pink */
}
```

### Visual Style
- Rounded corners everywhere (border-radius 24px+)
- Thick borders (3–4px) in darker shade of element colour
- Paper-cutout drop shadows
- Soft radial gradients, no harsh linear ones
- Everything bounces, floats, or pulses — maximum joy

---

## RESPONSIVE LAYOUT

- **Mobile-first.** Uses `100svh` (small viewport height) to account for browser chrome
- Safe area insets for iPhone notch and home bar: `env(safe-area-inset-top)`, `env(safe-area-inset-bottom)`
- All fonts, padding, scene heights use `clamp()` to scale between small and large screens
- **Desktop:** card renders centred as a phone-shaped card (max 440px wide, 820px tall) with drop shadow
- Decorative 🎈🎉🎊 emojis appear on either side of the card on desktop via `body::before` / `body::after`
- On very short screens (< 620px tall), the hero photo hides automatically
- Fixed buttons (⚙️ settings, 🔊 sound) stay within the card frame using `calc(50% - 210px)` positioning

---

## SPLASH SCREEN

When the app first loads, a full-screen dark navy splash displays:
1. Baby-themed floating emojis animate in (👶 🍼 🧸 🎈 🚗 🦁 🎀 🌈 🎂 🎁) with `twinkle` animation
2. **Abhiram's photo** (circular, gold border, zoom-in animation at 200ms)
3. Text fades in line by line:
   - `"Abhiram's Journey"` (white, Baloo 2, 32px bold)
   - `"...of the most magical 2 years."` (gold, 20px)
   - 🎂 emoji (48px)
4. `"🔊 Audio will start automatically"` note appears
5. **"Begin the Story! 🎂"** button appears (yellow gradient, bouncy)

Tapping the button **immediately initialises the Web Audio API** (this is the required user gesture for browsers), starts the background music, plays a welcome chime, and transitions to the card.

---

## GLOBAL APP STRUCTURE

```
#splash              ← Full-screen animated intro
#app
  ├── #progress-bar  ← Month badge + 24 heart dots + progress fill bar
  ├── #slide-wrap    ← Sliding card area (perspective 3D transitions)
  │     └── .slide → .card → [month-badge, slide-title, photo-hero, scene, caption, interact-zone]
  └── #nav           ← ← prev | Month X/24 counter | → next (disabled until slide complete)
#sound-btn           ← 🔊/🔇 fixed top-left (yellow when on)
#settings-btn        ← ⚙️ fixed top-right
#share-screen        ← Full-screen share overlay (shown after slide 24 complete)
#personal-panel      ← Slide-up personalisation drawer
#confetti-canvas     ← Full-screen canvas for particle confetti
```

### Navigation Rules
- Left ← and right → arrow buttons with spring-physics bounce on press
- **Right arrow is DISABLED until the current slide's interaction is completed**
- After 3 seconds with no interaction, the → button wiggles as a hint
- Swipe left/right gesture works on touch devices
- Keyboard arrow keys work on desktop
- Slide transitions: 3D page-flip (`rotateY`) with `cubic-bezier(.25,.8,.25,1)`

### Photo Upload System
- Every slide has a small dashed circular 📸 slot in the top-right of the card header
- Tapping it opens `<input type="file" accept="image/*">`
- Uploaded photo appears as a **full-width hero image** (160px tall, rounded corners) below the title
- The small circle also updates to show a thumbnail
- "🔄 Change" button overlays the hero photo
- On very short screens the hero auto-hides
- All photos stored in React/JS state (session only)

### Completion System
- `completeSlide(idx, emoji, delay=0)` — marks slide done, fires after `delay` ms
- Shows a brief overlay (emoji + "Amazing! Move to the next one →") for 1.8s then fades
- Hearts spawn from the scene
- → button turns gold with pulse animation
- Heart dot for that month fills in the progress bar

---

## PERSONALISATION PANEL (⚙️)

Slide-up drawer accessible anytime. Fields:
- **Baby's Name** (default: Abhiram) — used in progress bar, final slide
- **His First Word** (default: Thatha)
- **His Best Friend** (default: Stuffed Giraffe)
- Shows a read-only list of all 15 Month 23 favourites (hardcoded)
- Saved to `localStorage`

---

## AUDIO SYSTEM (Web Audio API — no external files)

### Background Music
`Happy Birthday to You` — full melody with correct rhythm, loops with a 1.2s pause between repeats. Uses `triangle` oscillators routed through a `bgmGain` node (volume 0.18). Scheduled via `AudioContext.currentTime` offsets for precise timing.

### Sound Toggle
- 🔊 button top-left, **ON by default** (starts on the Begin button tap)
- Every `tone()` and `noise()` call checks `audioCtx.state === 'suspended'` and calls `.resume()` if needed

### Core Audio Primitives
```javascript
tone(freq, dur, type, vol, startOffset, dest)  // oscillator envelope
noise(dur, vol, centerFreq, q, startOffset)     // filtered noise buffer
```

### Per-Interaction Sounds
| Slide | Sound |
|---|---|
| Month 1 — Soothe | `sndCry()` wah-wah sawtooth → `sndShush()` high noise on completion |
| Month 2 — Smile | Ascending tone per tap |
| Month 3 — Bubbles | `sndPop()` filtered noise burst + pitch drop |
| Month 4 — Tickle | `sndGiggle()` ascending chord |
| Month 5 — Roll | Thud per tap → `sndRollSpin()` pitch sweep |
| Month 6 — Food | `sndNom(foodIdx)` different texture per food |
| Month 7 — Tooth | `sndClick()` → `sndTooth()` chime sparkle |
| Month 8 — Crawl | `sndCrawlStep()` thud per tap |
| Month 9 — Wave | `sndWave()` 3-note melody |
| Month 10 — Words | `sndWord(word)` — unique melody per word (5 different sounds) |
| Month 11 — Slider | `sndRising(pct)` pitch rises as baby stands |
| Month 12 — Footsteps | `sndStep()` per footprint |
| Month 13 — Candle | `sndBlow()` breath noise |
| Month 13 — Cake cut | Sawtooth slash + noise per cut |
| Month 14 — Simon | `sndSimonBtn(idx)` — C D E G tones (261 329 392 523 Hz) |
| Month 15 — Calm | `sndCalm()` soothing chord |
| Month 16 — Park | `sndParkTap(item)` per activity; descending tones for slide |
| Month 17 — Bath splash | `sndSplash()` noise burst |
| Month 17 — Duck | 3 quick square tones (quack) |
| Month 17 — Soap | Bubble noise + high sine |
| Month 18 — TV | `sndTvBeep(btn)` per button → `sndTvSuccess()` fanfare |
| Month 19 — Reactions | `sndNoBuzzer()` / `sndCarExcite()` / `sndCalm()` |
| Month 20 — Peek | `sndPeek()` ascending reveal |
| Month 21 — Meter | `sndLoveMeter(pct)` sparse chime → `sndLoveMax()` arpeggio |
| Month 22 — Stars | `sndCollect()` 3-note collect |
| Month 23 — Gift | `sndGiftReveal()` random triangle pop |
| Month 24 — Candles | `sndBirthdayFanfare()` full 14-note melody + bass chord |
| All completions | `sndCompletion()` 6-note ascending chime |
| Slide transitions | `sndSlideTransition()` 2-note swoosh |

---

## THE 24 SLIDES — COMPLETE SPECIFICATION

### MONTH 1 — The First Cry 👶
**BG:** Peach gradient  
**Scene:** Large crying baby emoji  
**Interaction:** Tap the baby / "Soothe Him!" button 5 times. Face transitions: 😭→😥→😢→🥲→😌. Tears clear.  
**Completion:** 5 taps  
**Sound:** Cry sound each tap, shush on completion

---

### MONTH 2 — The First Smile 😊
**BG:** Sunshine yellow  
**Scene:** Baby face emoji starts neutral 😐  
**Interaction:** Tap face 4 times. Progresses: 😐→🙂→😊→😄→😁. Hearts spawn at max.  
**Completion:** 4 taps  
**Sound:** Ascending tone per tap

---

### MONTH 3 — The First Bath 🛁
**BG:** Sky blue  
**Scene:** 🛁 at bottom, 12 floating coloured bubbles  
**Interaction:** Tap each bubble to pop it (pop animation + sound). All 12 must be popped.  
**Completion:** 12 pops  
**Sound:** Filtered noise pop per bubble

---

### MONTH 4 — The First Laugh 😂
**BG:** Mint green  
**Scene:** Large baby emoji + 4 glowing tickle-spot circles (belly, feet, hand, chin)  
**Interaction:** Tap all 4 tickle spots. Baby face progresses: 😐→😄→😆→🤣→😂  
**Completion:** All 4 tapped  
**Sound:** Giggle chord per spot

---

### MONTH 5 — The First Roll 🎲
**BG:** Berry red  
**Scene:** Baby face 👶 + power bar  
**Interaction:** Tap "💪 POWER UP!" 10 times to fill bar. On completion: baby spins (CSS `spin` animation 1.2s) then lands on 🙃  
**Completion:** 10 taps (overlay delayed 1.4s for spin to play)  
**Sound:** Thud per tap, spin whoosh at 10

---

### MONTH 6 — First Solid Food 🍌
**BG:** Sunshine yellow  
**Scene:** Large baby face emoji 😐 + 3 food items below (🍌 🥔 🍎)  
**Interaction:** Tap each food. Baby reacts: 😄 (banana), 😮 (potato), 😖 (apple). Food item shrinks/disappears. Floating reaction text appears.  
**Completion:** All 3 foods tapped  
**Sound:** Different nom texture per food

---

### MONTH 7 — The First Tooth 🦷
**BG:** Lavender  
**Scene:** Baby face 😶 (mouth closed)  
**Interaction:** Tap "👆 Peek inside!" button 3 times. Staged reveal: 😶→🙂→😁→sparkle with 🦷✨ FIRST TOOTH! ✨🦷  
**Completion:** 3 taps  
**Sound:** Click each tap, chime sparkle on final reveal

---

### MONTH 8 — The First Crawl 🐛
**BG:** Mint green  
**Scene:** Green floor + checkered finish line on right. Baby 👶 starts at left edge.  
**Interaction:** Tap "▶ Go! Go! Go!" 15 times. Baby moves right with each tap (crawl animation). Reaches finish and becomes 🏆.  
**Completion:** 15 taps  
**Sound:** Crawl thud per tap

---

### MONTH 9 — First Wave Bye-Bye 👋
**BG:** Sky blue  
**Scene:** Large baby 👶 emoji  
**Interaction:** Tap "👋 Wave Back!" 3 times. Baby progresses: 👶→🙋→🙌. Shake animation per tap.  
**Completion:** 3 waves  
**Sound:** 3-note wave melody

---

### MONTH 10 — The First Word 🗣️
**BG:** Lavender  
**Scene:** Large 👶 baby face (reacts per word)  
**Buttons:** 5 word buttons: **Thatha · Caar · Noo · Mommy · Daddy**  
**Interaction:** ALL 5 buttons must be tapped. Each plays a unique sound and the baby shows a different reaction face (🥰😍😤😄😁). Button turns green when first tapped. Counter shows X/5.  
**Special:** Tapping "Thatha" triggers confetti + "The whole family went WILD! 🥹"  
**Completion:** All 5 words heard  
**Sounds:**
- Thatha: warm ascending sine melody
- Caar: sawtooth vroom + rising pitch
- Noo: descending protest notes
- Mommy: sweet gentle ascending
- Daddy: deep triangle tones

---

### MONTH 11 — First Pull to Stand 🧗
**BG:** Peach  
**Scene:** Large 🛋️ sofa (64px) + baby progression + vertical slider track  
**Slider thumb:** Shows 🤚 (baby hand gripping)  
**Interaction:** Drag the slider upward. Baby changes: 👶 sitting → 🤸 pulling → 🧗 climbing → 🧍‍♂️ standing (gold glow at 80%)  
**Label below baby:** "Sitting..." → "Pulling up..." → "Almost there!" → "Standing! 🌟"  
**Completion:** Slider at 95%+ → baby becomes 🙌 with "🎉 He stood up!"  
**Sound:** Rising pitch tone that gets higher as slider goes up

---

### MONTH 12 — The First Steps 👣
**BG:** Peach  
**Scene:** Warm background, 8 footprints in a zigzag path in the upper portion. Baby 👶 at bottom-left.  
**Interaction:** Tap footprints in order (left-right-left zigzag). Wrong tap = shake animation. Correct = footprint lights up, baby moves right along bottom.  
**Completion:** All 8 footprints in order  
**Sound:** Step thud per correct tap

---

### MONTH 13 — First Birthday 🎂
**BG:** Rainbow gradient  
**Scene Phase 1:** Big "2" number + baby name + 🎂 cake with two 🕯️ candle flames  
**Interaction Phase 1:** Tap both candles (can tap either order). Each candle flame gets dimmer then goes 💨. After both blown: fanfare plays, confetti fires, cut-phase reveals after 1.5s.  
**Scene Phase 2 (Cut the Cake):** Cake 🎂 + knife 🔪 + empty plate 🍽️  
**Interaction Phase 2:** Tap the knife 3 times:
- Tap 1: knife slashes, plate appears faintly — "First cut! ✂️"
- Tap 2: slice 🍰 appears on plate — "Another slice!"
- Tap 3: cake dims, full slice served, triple confetti, "🎉 Cake served! Happy Birthday Abhiram!" → Share screen opens  
**Completion:** 3 knife taps  
**Sound:** Birthday fanfare on candles, sawtooth slash per knife tap

---

### MONTH 14 — First Dance 💃
**BG:** Bubblegum  
**Scene:** Dancing baby 🕺 + hint text  
**Interaction:** Simon Says — 4-colour button sequence (🔴🔵🟡🟢 = C D E G notes). System flashes a 4-step pattern; user must repeat. Wrong = retry. The preview also plays the tones.  
**Completion:** Correct sequence entered  
**Sound:** Real musical tones per button (C=261Hz, D=329Hz, E=392Hz, G=523Hz)

---

### MONTH 15 — First Big Feelings 😤🥹
**BG:** Berry red  
**Scene:** Shaking 😤 baby emoji + random "Reason for tantrum" text (from a list of funny reasons)  
**Caption:** "Sometimes happy, sometimes cranky — always 100% expressive about it."  
**Interaction:** Tap "🤗 Calm Down" 8 times. Calm bar fills. Baby turns 😌 at 100%.  
**Completion:** 8 taps  
**Sound:** Soothing chord per tap

---

### MONTH 16 — First Time at the Park 🏡
**BG:** Mint green  
**Scene:** Sky + grass background. Three interactive areas:
1. **Swing (left):** Real wooden frame (two vertical bars + horizontal top bar), swing seat + ropes, baby 👶 sitting on it
2. **Slide (centre):** SVG-drawn orange ramp with ladder. Baby 👶 visible at the top, waiting
3. **Sandbox (right):** 🧒 + sandbox pit 🏖️  
**Interactions:**
- Tap swing → pendulum swing animation (rotate 25° → -20° → 10° → 0°) + "Whee! 🎉"
- Tap slide → baby animates diagonally down the SVG ramp (translate 46px,46px rotate 28°) with descending tones → "Wheeeee!! 🛝" → baby resets to top
- Tap sandbox → bounce + rotate + "Digging! 🏖️"  
**Completion:** All 3 activities tapped  
**Sound:** Different park sounds per activity

---

### MONTH 17 — Bath Time! 🛁
**BG:** Sky blue  
**Scene:** Inflated oval baby bath (pink border, blue water, rounded tub shape). Baby 👶 inside. Duck 🦆 on left, Soap 🧼 on right.  
**Three interactions (ALL required):**
1. **Tap bath** (8 times) → splash 💦 drops, baby bounces, counter fills
2. **Tap soap 🧼** → lather overlay (🫧×10) fades over baby, coloured bubble circles burst across water
3. **Tap duck 🦆** → duck rotates, baby shows 🥰, hearts ❤️💕💖 float up  
**Completion:** All three done  
**Sounds:** Splash noise, duck quack tones, bubble sound

---

### MONTH 18 — Nursery Rhymes 🎵🎶
**BG:** Rainbow gradient  
**Caption:** "Baby Shark, Wheels on the Bus, Johnny Johnny... on repeat. All. Day. Every. Day."  
**Scene:** Retro TV with static screen + 4 coloured channel buttons  
**Code:** Press 🔴🟡🔵🟢 in that order. Wrong sequence = reset + hint shown.  
**On success:** Screen turns sky blue, shows 🎵🎶 bouncing, "Baby Shark, do do do do! 🎶"  
**Completion:** Correct 4-button sequence  
**Sound:** Beep per button, fanfare on success

---

### MONTH 19 — First Words! 🗣️
**BG:** Berry red  
**Scene:** Baby emoji (changes per reaction)  
**Buttons (5, each triggers unique reaction):**
- 🤗 A hug → 🥰 face + "Aww a hug! 🥰" (love sound)
- 🚗 Go Carwalking → 🤩 SUPER excited + "🚗 YES!! CARWALKING!!" (excite fanfare) — green button
- 🥦 Vegetables → 😖 fussy + "🥦 Ewww NO!! 😖" (buzzer)
- 😴 More sleep → 😤 shake + "NOOOO!! 😤" (buzzer)
- 🛁 A bath → 😄 smile + "Bath time? 😄 YAY!" (ascending tones)  
**Completion:** All 5 buttons pressed  
**Sound:** Per-reaction sounds

---

### MONTH 20 — Best Friends 🐾👧
**BG:** Lavender  
**Caption:** "Nakshatra, the dog, and you — the trio that makes every day better."  
**Scene:** Three independent tappable characters side by side:
- 🐶 **Doggo** (left) — tap → rotates + "Woof woof! 🐶", dot turns green
- 🥚→🐣 **Kodipilla** (centre) — tap egg → egg tilts/shrinks, chick pops out + "Kodipilla! 🐣", resets after 850ms, dot turns yellow
- 👧 **Nakshatra** (right) — tap → waves as 🙋‍♀️ + "Nakshatra! 🌸", dot turns pink  
Each has a coloured tick dot below that fills on first tap.  
**Completion:** All 3 characters tapped  
**Sound:** Reveal ascending tones

---

### MONTH 21 — The Love Meter ❤️
**BG:** Bubblegum  
**Scene:** SVG semicircular gauge. Needle starts pointing left. Arc fills as needle moves right. Labels: Like 🥹 → Love ❤️ → ADORE 💕 → INFINITY ♾️  
**Interaction:** Drag anywhere on the SVG overlay div. Uses `setPointerCapture` for reliable cross-device drag. Pointer position mapped to SVG coordinates, angle calculated from centre (120,120).  
**At MAX (99%):** Love arpeggio plays, confetti fires, hearts spawn  
**Completion:** Needle reaches max  
**Sound:** Sparse chime during drag, full arpeggio at max

---

### MONTH 22 — First Adventures 🌍
**BG:** Sky blue  
**Scene:** Map background (sky blue/teal gradient) with 5 ⭐ stars scattered at fixed positions  
**Interaction:** Tap each star to collect it. Star turns 🌟, collect sound plays.  
**Completion:** All 5 stars collected  
**Note:** Star labels can be customised (default: adventure milestones)

---

### MONTH 23 — Things You Love ❤️‍🔥
**BG:** Rainbow gradient  
**Scene:** Floating 🎁 gift box at bottom. Chips appear above it as each favourite is revealed.  
**Favourites (15 items, hardcoded):** 🚗 Car · 🐶 Dog · 🚶 Carwalking · 🐦 Pigeons · 🧸 Toys · 🎵 Nursery Rhymes · 🤱 Mummy · 👨 Daddy · 👵 Ammamma · 👴 Thatha · 👨‍👩‍👦 Mamaiyya · 🍳 Omlette · 🍫 Chocolate · 😴 Sleeping · 🛁 Bath  
**Interaction:** Tap the gift box once per favourite. Each tap reveals one chip (slide-up animation). Gift box shrinks as items fill. At all 15: box becomes 🎊, confetti fires.  
**Completion:** All 15 favourites revealed  
**Sound:** Random triangle pop per reveal

---

### MONTH 24 — Happy 2nd Birthday! 🎂
**BG:** Rainbow gradient  
**Scene Phase 1 (Candles):**
- Giant "2" (gradient animated)
- Baby's name below
- 🎂 cake with two 🕯️ candles (each tappable)  
**Phase 1 Interaction:** Tap both candles. Flames dim and become 💨. Birthday fanfare plays, confetti fires. After 1.5s transitions to Phase 2.

**Scene Phase 2 (Cut the Cake):**
- Cake 🎂 + knife 🔪 + plate 🍽️  
**Phase 2 Interaction:** Tap knife 3 times:
- Tap 1: knife slashes, "First cut! ✂️", plate faintly appears
- Tap 2: "Another slice! 🍰", slice appears on plate
- Tap 3: "🎉 Cake served! Happy Birthday Abhiram!", triple confetti → Share screen opens automatically  

**Sound:** Birthday fanfare (14-note melody + bass chord), sawtooth slash per knife tap

---

## SHARE SCREEN

Triggered automatically after slide 24 completes (cutting the cake). Full-screen dark navy overlay with:
- Floating birthday emojis raining down (CSS `floatDown` animation, 20 elements)
- White card with 🎂 + "Abhiram Turns 2! 🎉"
- **Editable textarea** pre-filled with:
  > "🎂 Abhiram Turns 2 Today! 🎉\nTap to open his interactive birthday card and wish him a very Happy Birthday! 🥳\nPlay through 24 months of his journey — and don't miss the cake! 🍰"
- **📤 Share to Apps** — `navigator.share()` native sheet (hidden if not supported)
- **💬 WhatsApp** — `https://wa.me/?text=` with encoded message + URL
- **🔗 Copy Link** — `navigator.clipboard.writeText()` + "✅ Link copied!" confirmation
- **💌 SMS** — `sms:?body=` with encoded message
- **← Back to the card** button

---

## STATE MANAGEMENT

```javascript
const state = {
  current: 0,                          // active slide index
  completed: new Array(24).fill(false), // completion flags
  photos: new Array(24).fill(null),     // base64 DataURLs
  name: 'Abhiram',
  firstWord: 'Thatha',
  bestFriend: 'Stuffed Giraffe',
  favs: [...15 items...],              // Month 23 favourites
  activePhotoSlot: 0,
  transitioning: false
};
```

All personalisation saved to `localStorage`. Photos stored in session state only (not persisted — too large).

---

## KEY HELPER FUNCTIONS

```javascript
completeSlide(idx, emoji, delay=0)     // marks complete, shows overlay after delay
spawnConfetti()                         // 120-particle canvas confetti
spawnHearts(container)                  // 5 floating hearts from scene
popText(scene, text)                    // floating speech bubble
spawnPopText2(text, idx)               // finale-specific floating text
buildSlideHTML(idx, sd)                 // generates full slide HTML
buildScene(idx, sd)                     // generates scene HTML per type
buildInteract(idx, sd)                  // generates interact zone HTML per type
setupInteraction(idx, sd)               // post-render JS setup (bubbles, simon, meter, etc.)
renderSlide(idx, fromDir)               // creates, appends, and transitions a slide
transitionSlide(from, to, dir)          // handles the 3D page-flip transition
navigate(dir)                           // handles prev/next with completion guard
```

---

## SLIDE TYPES REFERENCE

| Type | Month | Mechanic |
|---|---|---|
| `soothe` | 1 | Tap counter, emoji progression |
| `smile` | 2 | Tap counter, emoji progression |
| `bubbles` | 3 | Click-to-pop, Set tracking |
| `tickle` | 4 | Multi-zone tap, Set tracking |
| `power` | 5 | Rapid tap + progress bar + spin |
| `food` | 6 | Tap-to-feed, per-item reaction |
| `reveal` | 7 | Staged 3-tap reveal |
| `crawl` | 8 | Rapid tap = baby moves across screen |
| `wave` | 9 | Tap counter, emoji progression |
| `word` | 10 | 5 buttons all must be pressed, per-sound |
| `slider` | 11 | Vertical drag with pointer capture |
| `footsteps` | 12 | Tap-in-order sequence |
| `candle1` | 13 | Two-phase: blow + cut |
| `simon` | 14 | Simon Says sequence memory game |
| `tantrum` | 15 | Tap counter + calm bar |
| `park` | 16 | 3 independent tap zones |
| `splash` | 17 | 3 independent interactions (bath + duck + soap) |
| `tv` | 18 | 4-button code entry |
| `no` | 19 | 5 buttons each with unique reaction type |
| `peekaboo` | 20 | 3 independent tappable characters |
| `meter` | 21 | SVG drag with pointer capture + angle math |
| `map` | 22 | 5 tap-to-collect stars |
| `giftbox` | 23 | 15-tap sequential chip reveal |
| `finale` | 24 | Two-phase: candles + knife cuts |

---

## DEPLOYMENT

Single file — drop `index.html` onto **Netlify Drop** (netlify.com/drop) for instant free hosting. No build step, no dependencies, no backend.

Alternatively: GitHub Pages — push as `index.html` to any repo with Pages enabled.

---

*Built with love for Abhiram's 2nd birthday. 🎂*
