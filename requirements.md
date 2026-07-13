# ReTint — Requirements & Installation

To run ReTint's extraction and grafting skills locally, you need the following dependencies:

## Prerequisites
- **Node.js** (v18.0.0 or higher)
- **npm** (v9.0.0 or higher)
- **Git** (installed and configured)

## Package Dependencies
The project uses the following npm packages:
- **Playwright** (for browser automation and DOM style extraction)

## Setup Instructions

1. Initialize and install dependencies:
   ```bash
   npm init -y
   npm install playwright
   ```

2. Install the Chromium browser binaries for Playwright:
   ```bash
   npx playwright install chromium
   ```
