# MSG Jumbotron Forced Perspective Basketball — Design

**Date:** 2026-03-22
**Approach:** Anamorphic Camera (camera-below-looking-up)

## Overview

A real-time Three.js web app simulating the MSG center-hung jumbotron. All 4 screens display identical content. A forced perspective illusion makes a bouncing basketball appear to pop out of the screen when viewed from courtside/lower bowl seats (looking up at the jumbotron).

## Scene Architecture

A virtual "box" — the screen is a window into a room viewed from below:

- **Floor plane** — hardwood basketball court with Knicks center court logo
- **3 visible walls** — dark walls (left, right, back) framing the scene; front wall is the screen opening; ceiling is open/dark
- **Basketball** — orange textured sphere bouncing left-to-right then right-to-left (ping-pong)
- **Shadow** — soft dark ellipse on the court floor below the ball, scales with ball height

## Camera Setup

- Perspective camera positioned below scene center, angled up ~60-70°
- FOV tuned so box walls create strong converging lines (sells the 3D illusion)
- Camera locked — no orbit controls

## Animation

- Ball follows parabolic arc (gravity simulation) bouncing between left and right walls
- Slight energy loss per bounce, resets to full energy at loop start
- Shadow: ellipse stretches/shrinks with ball height, tracks ball X position
- Smooth ping-pong loop: left→right, then right→left

## Visuals / Branding

- **Knicks orange:** #F58426
- **Knicks blue:** #006BB6
- **Knicks silver:** #BEC0C2
- Court floor: hardwood look with Knicks logo at center (procedural or placeholder)
- Ball: orange with black seam lines
- Walls: dark blue/black with subtle gradient
- Background: pure black (matches arena darkness)

## Tech Stack

- Single HTML file, inline Three.js via CDN
- 16:9 canvas, fullscreen-capable
- No dependencies beyond Three.js
- Real-time 60fps target

## Delivery

- All 4 jumbotron screens show identical content
- Optimized for courtside/lower bowl viewing angle (looking up)
- Real-time web app, controllable live at venue
