# CONTRIBUTING

We're building the definitive collection of public `ai-catalog.json` files — the "ARD Catalog Bible."

## How to Contribute

### 🆕 Add a New Catalog

Found a website that publishes `/.well-known/ai-catalog.json`? Contributions welcome!

1. Fork this repository
2. Create a directory `catalogs/<publisher-name>/`
3. Add the raw `ai-catalog.json` file (preserve original formatting)
4. Add a `meta.json` with these fields:
   ```json
   {
     "source": "https://example.com/.well-known/ai-catalog.json",
     "collected": "2026-06-26",
     "notes": "Brief notes about this publisher"
   }
   ```
5. Update `INDEX.md` with the new entry
6. Submit a Pull Request

### 📝 Report a Broken Catalog

If a catalog URL goes offline or returns an error, [open an issue](https://github.com/huangxn29/ard-catalogs/issues) with:
- The publisher name and URL
- The HTTP status or error you received
- When you last confirmed it was working (if known)

### 📢 Spread the Word

- **Star** this repo to help others discover it
- **Follow [@huangxn29 on X](https://x.com/huangxn29)** for ARD ecosystem updates
- Share the repo with your network if you find ARD interesting

## Guidelines

- Only accept **publicly accessible** `ai-catalog.json` files (no authentication required)
- Preserve the **original JSON format** without modification
- If a catalog goes offline, update its status rather than removing it immediately

## Questions?

Open an issue or reach out on X at [@huangxn29](https://x.com/huangxn29).
