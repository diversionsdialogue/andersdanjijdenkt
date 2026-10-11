# andersdanjijdenkt.nl: statische site live zetten

Statische export van WordPress (Simply Static). Geen build-stap, geen PHP.
Route: GitHub -> Ploi -> Hetzner. Formulier: browser -> `/api/formulier` -> nginx (+secret) -> n8n -> Mailjet.

## Wat er t.o.v. de ruwe export is aangepast

- WordPress-restpagina's verwijderd (`hello-world`, auteurs, `uncategorized`) en in nginx 301'd.
- Canonical, `og:url`, `og:image`, `twitter:image`, JSON-LD en sitemaps zijn absoluut (`https://andersdanjijdenkt.nl/...`).
- `sitemap.xml` verwijst alleen naar `page-sitemap.xml` (5 pagina's). `robots.txt` en `llms.txt` opgeschoond.
- WordPress-koppen (`EditURI`, `shortlink`) weg; footertelefoonlink was `tel:123-456-7890`, nu `tel:+31552032193`.
- Contactformulier (`/over-ons/#contact`) post JSON naar `/api/formulier`, met honeypot en invultijd. Elementor-Pro-handler staat uit voor dit formulier (`data-widget_type="form.static"`), anders post het naar het niet-bestaande `admin-ajax.php`.
- Ongeldige `pattern` op het telefoonveld vervangen (de Elementor-variant werd door Chrome genegeerd).
- `404.html` toegevoegd (noindex).

## Voor go-live

### 1. Server (Ploi)
- [ ] Site `andersdanjijdenkt.nl` aanmaken, repo koppelen, branch `main`, geen build-script.
- [ ] Secret genereren: `openssl rand -hex 32` (wachtwoordmanager).
- [ ] `deploy/nginx-andersdanjijdenkt.conf` vergelijken met Ploi's gegenereerde config (include-paden, root), `[geheim]` invullen, in zijn geheel plakken (skill `ploi-nginx-astro`).
- [ ] `nginx -t` en reload.

### 2. n8n
- [ ] `deploy/n8n-formulier-workflow.json` importeren.
- [ ] Credential "Header Auth": header `X-Webhook-Secret`, waarde = secret van stap 1; kiezen op de Webhook-node.
- [ ] Credential "Mailjet Email API" kiezen op de Mailjet-node.
- [ ] Data Table `formulierInzendingen-andersdanjijdenkt` aanmaken met kolommen: formulier, voornaam, achternaam, organisatie, email, telefoon, bericht (allemaal string); kiezen in "Bewaar inzending".
- [ ] Error workflow instellen, workflow activeren.
- [ ] Afzender `info@generationjourney.nl` en ontvanger `info@diversions.nl` zijn aangenomen (zoals bij generationjourney.nl). Pas aan als dat niet klopt.

### 3. Tests (na deploy op testdomein)
- [ ] Echte inzending: succesmelding, rij in Data Table, mail ontvangen.
- [ ] Honeypot gevuld: succesmelding, maar geen rij en geen mail.
- [ ] `curl` zonder verplicht veld: 400 met details.
- [ ] Directe POST naar de n8n-webhook zonder secret: geweigerd.
- [ ] >5 POSTs per minuut: 429.
- [ ] `/hello-world`-URL's, `/author/redactie/`, `/post-sitemap.xml`: 301; onbekende URL: 404.

### 4. DNS-omzetting
- [ ] Het domein draait nu op een LiteSpeed-host. Noteer vooraf MX/SPF/DKIM en andere records en neem ze over als er mail op het domein loopt.
- [ ] A-record naar Hetzner, daarna pas SSL aanvragen in Ploi.
- [ ] Google Search Console: sitemap `https://andersdanjijdenkt.nl/sitemap.xml` opnieuw indienen.

### 5. Administratie
- [ ] Verwerking formuliergegevens opnemen in het privacy statement.
- [ ] Google-tag `GT-WRF7M5PW` draait zonder cookiebanner (zat al zo in WordPress). Beslissing nodig.
