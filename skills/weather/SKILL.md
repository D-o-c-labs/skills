---
name: weather
description: Weather forecast skill using configurable URL-template providers and wttr.in fallback.
metadata:
  {
    "openclaw":
      {
        "emoji": "🌦️",
        "os": ["linux", "macos"],
        "requires": { "bins": ["python3"] },
        "install":
          [
            {
              "id": "apt",
              "kind": "apt",
              "packages": ["python3", "python3-requests"],
              "bins": ["python3"],
              "label": "Install Python 3 with requests via apt",
            },
          ],
      },
  }
---

# Weather

This skill uses `SKILL_WEATHER_*` environment variables for provider selection and URL-template configuration.

- `SKILL_WEATHER_PROVIDER` defaults to `meteoam`.
- `SKILL_WEATHER_EXTRACTOR_ORDER` defaults to `jina,tavily`.
- `SKILL_WEATHER_CUSTOM_<NAME>_URL` defines or overrides a URL-template provider and must include `{city}`.
- `SKILL_WEATHER_CUSTOM_<NAME>_EXTRACTORS` overrides extractor priority for that provider.
- `JINA_API_KEY` and `TAVILY_API_KEY` are shared credentials rather than skill-prefixed variables.
- Built-in URL-template providers: `meteoam`, `3bmeteo`.
- Built-in API provider: `wttr`.

## Usage

```bash
python3 skills/weather/scripts/weather.py Rome
python3 skills/weather/scripts/weather.py Rome --provider meteoam
python3 skills/weather/scripts/weather.py "New York" --provider 3bmeteo
python3 skills/weather/scripts/weather.py Rome --provider wttr
```

The script prints JSON to stdout. For URL-template providers, the returned `text` field is the raw extractor output. For `wttr`, the `text` field is the raw JSON response body from `wttr.in`.

## Provider Rules

- The location argument is required.
- Provider URLs receive the location lowercased and URL-encoded.
- URL-template providers do not fall back automatically. If the selected provider fails, the script returns an error JSON object and the agent should decide whether to retry with another provider.

## Extractors

URL-template providers use web extraction in configured priority order:

- `jina` reads `https://r.jina.ai/{url}` and returns markdown.
- `tavily` calls `https://api.tavily.com/extract` and returns `results[0].raw_content` when available.

Behavior:

- Missing extractor API keys are skipped immediately.
- Each configured extractor retries up to 3 times with a 5 second delay before moving to the next extractor.
- Unknown extractor names are ignored.

## Output Shape

Success:

```json
{
  "provider": "meteoam",
  "location": "Rome",
  "url": "https://www.meteoam.it/it/meteo-citta/rome",
  "fetchedAt": "2026-04-11T09:00:00Z",
  "text": "## Rome\n\n15°C, partly cloudy..."
}
```

Error:

```json
{
  "error": "All extraction methods failed",
  "provider": "meteoam",
  "tried": ["jina", "tavily"],
  "location": "Rome"
}
```
