# Magazine Registry — Prime Public Media AG

Prime Public Media AG publishes 15 medical specialty magazines. Each has its own production specification under this directory. Skills (`/kb-cme-draft`, `/kb-issue-plan`, `/kb-editorial`, `/kb-issue-compose`, etc.) accept a `--magazine <slug>` flag; if omitted they use `default_magazine` from `~/.claude/kb-config.json`.

## Active specifications

| Slug | Magazine | Specialty | Spec file |
|---|---|---|---|
| `info-np` | InFo Neurologie & Psychiatrie | Neurology + Psychiatry | [MAGAZINE-INFO-NP.md](MAGAZINE-INFO-NP.md) |

## Pending specifications (other 14 specialty titles)

Each new title is reverse-engineered from a sample of past issues using the same 4-agent process used for `info-np`:
1. Magazine structure & editorial blueprint
2. CME deep dive
3. Pharma content & doctor's choice
4. Tone of voice & editorial standards

After the analysis, a `MAGAZINE-<slug>.md` file is written to this directory and registered in the table above.

## Cross-magazine conventions (apply unless a magazine spec overrides)

- Publisher: Prime Public Media AG, Neugasse 10, 8005 Zürich
- Default language: German (Schweiz). Swiss orthography (`ss` not `ß`, Helvetisms permitted).
- Default audience country tilt: Schweiz primary, DACH secondary (German-DE contributors marked with `(D)` tag in attributions).
- Decimal separator: comma (`1,5 mg`); thousands separator: thin space (`100 000`); confidence interval: `95%-KI`.
- Anführungszeichen: «...» (Guillemets).
- Drug names: INN-first in editorial content; brand-first only in clearly marked sponsored content with ®/™ + Kurzfachinformation.
- CME credits via `medizinonline.com` (Prime Public Media's accreditation platform), not directly via SIWF/EACCME/BÄK.
- Sponsored content labels: `PUBLIREPORTAGE`, `SONDERREPORT`, `MARKT & MEDIZIN`. Always uppercase, top of page.
- Page count must be divisible by 4 (print constraint).
- Editorial-commercial firewall: brand-first wording is the cleanest signal of sponsored content; editorial body uses INN.

A magazine-specific spec overrides these only where explicitly stated. The `info-np` spec is the canonical model.
