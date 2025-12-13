# BWS Workout Tracker

Progressive Web App (PWA) for tracking Built With Science (BWS) Beginner Program workouts with offline support and progression tracking.

**Live App:** [https://pranlawate.github.io/gym-tracker/](https://pranlawate.github.io/gym-tracker/)

---

## Features

### Workout Tracking
- **Phase 1 & Phase 2 Programs** - Full body (3x/week) and 5-day split workouts
- **Offline Support** - Install as PWA, works without internet
- **Exercise Videos** - 69 local MP4 demonstrations for proper form
- **Progressive Overload** - Auto-suggestions based on last workout performance
- **History Tracking** - Last 8 workouts per exercise with trend analysis
- **RPE Logging** - Rate of Perceived Exertion (1-10 scale)

### Smart Progression
- **Color-coded Alerts:**
  - 🟢 Green: Hit rep target → increase weight
  - 🟡 Yellow: Plateau detected → try deload/alternatives
  - 🔴 Red: Regression detected → check recovery
  - 🔵 Blue: Normal progress

### Exercise Alternatives
- **Dropdown Menus** - Easier/Harder/Alternate options for every exercise
- **Equipment Substitutions** - Dumbbell, band, and bodyweight alternatives
- **Video References** - YouTube links for SHRED HOME exercises

---

## Installation

### Use as Web App
1. Visit [https://pranlawate.github.io/gym-tracker/](https://pranlawate.github.io/gym-tracker/)
2. Works immediately in any browser

### Install as PWA (Recommended)
**Mobile (iOS/Android):**
1. Open app in Safari/Chrome
2. Tap Share → "Add to Home Screen"
3. App installs with offline support

**Desktop (Chrome/Edge):**
1. Visit app URL
2. Click install icon in address bar
3. App opens in standalone window

---

## Quick Start

### First Workout
1. Select **Phase 1** or **Phase 2** from top menu
2. Choose workout day (e.g., Monday)
3. For each exercise:
   - Click **"Load Last Workout"** for progression suggestions
   - Enter **Weight**, **Reps**, **RPE** for each set
   - Click **"Save Workout"** when done

### View Progress
- Click **📈 History** button next to any exercise
- See last 8 workouts with trend indicators (⬆️⬇️➡️)
- Review best weight and average RPE

### Change Exercises
- Use dropdown menu next to exercise title
- Select **Easier/Harder/Alternate** options
- Choice saves automatically for future workouts

---

## Project Structure

```
/
├── index.html                    # Main PWA (Phase 1 & 2 workouts)
├── bws-workout-plan.md           # Workout program reference guide
├── bws-course-notes.md           # Complete course documentation
├── workout-log-protocol.md       # Workout session logging protocol
├── docs/
│   ├── Exercise-Alternatives-Reference.md  # YouTube videos & written alternatives
│   ├── PROGRESSION-TRACKING-GUIDE.md       # Progression strategies for beginners
│   ├── VIDEO-LIBRARY.md                    # Catalog of 69 exercise videos
│   ├── Gym-Trainer-vs-BWS-Comparison.md    # Personal trainer program analysis
│   └── Google-Sheets-Setup.md              # Apps Script for horizontal layout sync
├── Sources/                      # BWS official PDFs (9 files)
└── Exercise-Videos/              # 69 MP4 exercise demonstrations
```

---

## Documentation

### For Users
- **[bws-workout-plan.md](bws-workout-plan.md)** - Main workout program (Phase 2 focus, beginner-friendly)
- **[PROGRESSION-TRACKING-GUIDE.md](docs/PROGRESSION-TRACKING-GUIDE.md)** - How to progress when stuck (2.5kg→5kg gap, tempo progression, microloading)
- **[Exercise-Alternatives-Reference.md](docs/Exercise-Alternatives-Reference.md)** - Home workout alternatives with video links
- **[VIDEO-LIBRARY.md](docs/VIDEO-LIBRARY.md)** - Complete video catalog by muscle group

### For Course Content
- **[bws-course-notes.md](bws-course-notes.md)** - Complete BWS course notes (nutrition, training, progression, strength goals)

---

## Tech Stack

- **Frontend:** Vanilla JavaScript (no frameworks)
- **Storage:** localStorage (client-side only, no server)
- **PWA:** Service Worker for offline caching
- **Videos:** Local MP4 files (650MB total)
- **Hosting:** GitHub Pages

---

## Beginner Tips

### Stuck at Light Weights?
If you can't progress from 2.5kg to 5kg dumbbells (100% increase):

1. **Tempo Progression** - Use 3-second eccentric (lowering) phase at 2.5kg
2. **Microloading** - Add wrist weights (0.5-1kg increments)
3. **Barbell Alternative** - Fixed barbells have smaller jumps (10kg→12kg→15kg)
4. **Increase Reps First** - Build to 15 reps at 2.5kg before trying 5kg

See [PROGRESSION-TRACKING-GUIDE.md](docs/PROGRESSION-TRACKING-GUIDE.md) for detailed strategies.

---

## License

Personal project for tracking Built With Science (BWS) Beginner Program workouts.
Course content and PDFs © Built With Science / Jeremy Ethier.

---

## Credits

- **Program:** Built With Science Beginner Program by Jeremy Ethier
- **PWA Development:** Personal project
- **Exercise Videos:** BWS SHRED HOME Program

---

**Last Updated:** 2025-12-13
