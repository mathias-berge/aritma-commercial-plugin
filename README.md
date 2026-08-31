# Aritma Commercial (plugin)

Denne pluginen samler Aritmas interne Claude-skills:

- `skills/aritma-compliance-agent` — besvarer sikkerhets-/compliance-spørreskjemaer fra Aritmas Notion-kunnskapsbase.
- `skills/aritma-marketing-agent` — markedsføringsstøtte, kundecase-innhold, marketing-rapportering. Bruker Notion (Marketing Agent – Knowledge Base, inkl. Customer Case Library), HubSpot og Slack.
- `skills/aritma-sales-agent` — kundevendt salgstekst, prising, konkurrentinfo og deal-/pipeline-forecast. Bruker Notion (Sales Agent – Knowledge Base) og HubSpot.
- `skills/aritma-pptx` — brand-riktige PowerPoint-presentasjoner for Aritma.

## Slik tar dere den i bruk

### Alternativ 1: Del som marketplace via git (anbefalt)

1. Push denne mappen til et internt git-repo (kan være privat).
2. Del repo-URL-en med teamet.
3. Hver kollega går til Settings → Plugins → Add marketplace i Claude-appen, limer inn repo-URL-en, og installerer `aritma-commercial`.

### Alternativ 2: Del skillsene enkeltvis

Hver undermappe i `skills/` er også en frittstående skill og kan installeres/lastes opp separat hvis dere ikke vil bruke marketplace-flyten ennå.

## Viktig: tilkoblinger (Notion, HubSpot, Slack)

Skillsene her forutsetter at brukeren selv har koblet til Notion, HubSpot og Slack under egne Settings → Connectors i Claude-appen (OAuth-basert, per bruker). Dette følger IKKE automatisk med pluginen — hver kollega må koble til disse selv én gang, akkurat som man gjør i dag. Pluginen pakker kun sammen selve instruksjonene (skillsene), ikke tilgangen til dataene.

## Vedlikehold

Når en av skillsene oppdateres (f.eks. ny versjon av Compliance Agent), oppdater `SKILL.md`-filen i riktig undermappe og øk versjonsnummeret i `.claude-plugin/plugin.json`. Push til git-repoet — kollegaer som har lagt til marketplace-en får oppdateringen automatisk neste gang de sjekker for oppdateringer.
