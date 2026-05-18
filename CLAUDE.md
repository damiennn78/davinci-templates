# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DaVinci Resolve Fusion template package system. 

## Repository Architecture

```
Template/
  <NOM_TEMPLATE>/             ← one folder per template (source editable)
    Edit/                     ← zipped to produce the .drfx
      Titles/
        Assets/
        <NOM_TEMPLATE>.setting ← DaVinci Resolve Fusion composition
    edit.drfx       ← packaged output (ZIP of Edit/ renamed .drfx)
  CLAUDE.md
```

**Rule**: every subfolder directly under `Template/` is a template.
Each can have some assets located at `Edit/Titles/Assets/`.


