---
description: "Architecture constraints for Excalidraw"
globs: packages/excalidraw/**
alwaysApply: false
---

# Excalidraw Architecture

## State Management

- Custom state via actionManager — NOT Redux/Zustand/MobX
- State updates: actionManager.dispatch() ONLY
- State type: AppState (packages/excalidraw/types.ts)

## Rendering

- Canvas 2D rendering — NOT React DOM for drawing
- Render pipeline: Scene → renderScene() → canvas context
- DO NOT use react-konva, fabric.js, pixi.js

## Dependencies

- No new npm packages without explicit approval
- Check packages/utils/ before adding external helpers
