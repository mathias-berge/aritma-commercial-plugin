# Aritma Commercial (plugin)

Denne pluginen samler Aritmas interne Claude-skills:

- `skills/aritma-compliance-agent` — besvarer sikkerhets-/compliance-spørreskjemaer fra Aritmas Notion-kunnskapsbase.
- `skills/aritma-marketing-agent` — markedsføringsstøtte, kundecase-innhold, marketing-rapportering. Bruker Notion (Marketing Agent – Knowledge Base, inkl. Customer Case Library), HubSpot og Slack.
- `skills/aritma-sales-agent` — kundevendt salgstekst, prising, konkurrentinfo og deal-/pipeline-forecast. Bruker Notion (Sales Agent – Knowledge Base) og HubSpot.
- `skills/aritma-solution-architect-agent` — tekniske avklaringer om produkt, filformater, bankdetaljer og ERP-integrasjoner for commercial-teamet. Søker fritt i hele Notion (ingen egen samlet kunnskapsbase-side) og bruker HubSpot for kunde-/dealkontekst.
- `skills/aritma-pptx` — brand-riktige PowerPoint-presentasjoner for Aritma.

## Slik tar dere den i bruk

Denne pluginen distribueres via et internt, privat git-repo. Du trenger ikke sette opp noe selv:

1. Be Mathias Berge om URL-en til repoet og eventuell tilgangsnøkkel (siden repoet er privat).
2. Gå til Settings → Plugins → Add marketplace i Claude-appen, og lim inn repo-URL-en du fikk.
3. Installer `aritma-commercial` fra marketplace-listen.

Ved fremtidige oppdateringer: du trenger ikke gjøre noe nytt oppsett — bare sjekk for oppdateringer i Plugins-panelet, eller fjern og legg til pluginen på nytt dersom oppdateringen ikke dukker opp automatisk.

## Viktig: tilkoblinger (Notion, HubSpot, Slack)

Skillsene her forutsetter at brukeren selv har koblet til Notion, HubSpot og Slack under egne Settings → Connectors i Claude-appen (OAuth-basert, per bruker). Dette følger IKKE automatisk med pluginen — hver kollega må koble til disse selv én gang, akkurat som man gjør i dag. Pluginen pakker kun sammen selve instruksjonene (skillsene), ikke tilgangen til dataene.

## Vedlikehold

Når en av skillsene oppdateres (f.eks. ny versjon av Compliance Agent), oppdater `SKILL.md`-filen i riktig undermappe og øk versjonsnummeret i `.claude-plugin/plugin.json`. Push til git-repoet — kollegaer som har lagt til marketplace-en får oppdateringen automatisk neste gang de sjekker for oppdateringer.
