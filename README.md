# skills

Public OpenClaw skills with configuration provided through environment variables.

## Contract

- Variable names follow `SKILL_<NAME>_<FIELD>` unless a shared credential is intentionally reused.
- Skill-specific usage details live in the relevant `SKILL.md`.

## Skills

### Weather

Path: `skills/weather/SKILL.md`

| Variable | Required | Default | Notes |
| --- | --- | --- | --- |
| `SKILL_WEATHER_PROVIDER` | no | `meteoam` | Default provider name |
| `SKILL_WEATHER_EXTRACTOR_ORDER` | no | `jina,tavily` | Global extractor priority for URL-template providers |
| `SKILL_WEATHER_CUSTOM_<NAME>_URL` | no | none | Defines or overrides a URL-template provider; must include `{city}` |
| `SKILL_WEATHER_CUSTOM_<NAME>_EXTRACTORS` | no | none | Per-provider extractor order override |
| `JINA_API_KEY` | no* | none | Shared credential for the Jina Reader extractor |
| `TAVILY_API_KEY` | no* | none | Shared credential for the Tavily Extract extractor |

\* At least one extractor API key is required when using URL-template providers such as `meteoam`, `3bmeteo`, or any custom site. `wttr` does not require credentials.

Built-in providers:

- `meteoam` -> `https://www.meteoam.it/it/meteo-citta/{city}`
- `3bmeteo` -> `https://www.3bmeteo.com/meteo/{city}`
- `wttr` -> `https://wttr.in/{city}?format=j1`

## Notes

- `JINA_API_KEY` and `TAVILY_API_KEY` are an intentional exception to the `SKILL_<NAME>_<FIELD>` convention because they are shared credentials that other tools may also use.
