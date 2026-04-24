# AFNOR XP Z12 Suite — Journal de projet

## Déploiement
- **GitHub** : github.com/dav06200ghi-lgtm/afnor-suite (public, branche main)
- **Vercel** : afnor-suite.vercel.app
- **Token Git** : [TOKEN_GITHUB]
- **Repo local** : /home/claude/repo_push/
- **Babel validator** : /home/claude/babel_test/test_parse.js
- **Stack** : HTML statique React/Babel CDN, api/chat.js proxy Anthropic
- **Fichier principal** : /home/claude/repo_push/index.html

## Contexte métier
- **David Ghiglione** — Directeur Financier, Coty Monaco / entités françaises
- **Projet** : Mise en conformité e-invoicing / e-reporting France 2026
- **ERP** : SAP S/4HANA
- **PDP** : Pagero (AR) + Tradeshift (AP)
- **Interlocuteurs** : Edgar Palau (e-Invoicing AP Lead), Laurent Malzac (Pagero FR), Valérie Charpentier

## Architecture app (onglets)
`📚 BT Fields` | `📊 E-Reporting` | `🔄 ILC` | `🗂️ Use Cases` | `🔬 Validator` | `⚖️ Rules` | `🏢 Coty FR` | `🔒 BFR Rules` | `📄 PUF Pagero` | `🌐 Flux` | `📅 Calendrier` | `🔍 Recherche` | `🤖 Expert IA`

## Sources intégrées
- XP_Z12-012_Annexe_A_2026_V1_3.xlsx — norme AFNOR
- ENG_XP_Z12-014_USE_CASES_Annex_A_V1_3.pdf — cas d'usage
- Coty_FR_tracker_22042026.xlsx — entités + cadres Coty
- e-reporing-flux10.xlsx — 155 champs TT/TG/TB DGFiP
- Copie_de_Best_practice_content_France.xlsx (v1.13) — Pagero BP
- Copie_de_Best_practice_content_France_Lifecycle.xlsx (v1.6)
- 16 PUF ApplicationResponse XML (UC4/10/13/14/19b/22a/26/34/40/41 + Generics)
- PUF e-reporting XML (POS B2C, B2Bi, Bi2B)

## États React clés
```js
const [tab,setTab]=useState("bt");
const [selBT,setSelBT]=useState(null);
const [grp,setGrp]=useState("all"); // section BT
const [sc,setSc]=useState("all"); // scope ei/cgi
const [req,setReq]=useState("all");
const [q,setQ]=useState('');
const [erQ,setErQ]=useState('');
const [erSc,setErSc]=useState('all');
const [erSort,setErSort]=useState('id');
const [erSortDir,setErSortDir]=useState('asc');
const [selER,setSelER]=useState(null);
const [selILC,setSelILC]=useState(ILC[0]);
const [ilcView,setIlcView]=useState('statuts'); // 'statuts' | 'motif'
const [ilcFam,setIlcFam]=useState('all');
const [pufType,setPufType]=useState('sales_b2bi');
const [lcEx,setLcEx]=useState(PUF_LC_EXAMPLES[0]);
const [bfrDoc,setBfrDoc]=useState('all');
const [bfrQ,setBfrQ]=useState('');
const [bfrCat,setBfrCat]=useState('all');
const [showLCRef,setShowLCRef]=useState(false);
```

## Données clés

### BT Fields — 210 entrées (169 BT + 41 EXT-FR-FE)
- `puf_req:` M/CM/O obligation Pagero Best Practice v1.13
- `puf_path:` chemin PUF Pagero (XPath)
- `puf_note:` commentaire Pagero
- `puf_path_note:` note PUF complémentaire
- `tt_er:` code TT e-reporting correspondant
- `idoc:/vim_field:/req_sap:/sap_note:` mapping SAP Coty
- `coty_note:` décision projet Coty (ex: BT-58 T&E)
- `dual_note:` BT-23 dual-rôle ProfileID vs @name
- `cadre:true` sur BT-23 → bloc Cadre Facturation
- `section:"Extensions"` sur EXT-FR-FE

Filtres section: all/header/seller/buyer/delivery/payment/allowcharges/amounts/vat/lines/line_allowcharges/line_items/Extensions

### E-Reporting — 155 entrées TT/TG/TB
- Colonnes: CODE | PUF/XML | DÉSIGNATION | STATUS | AR/AP | FLUX | BT | DGFiP XML
- `path:` chemin XML DGFiP (/ReportDocument/... ou /TransactionsReport/...)
- `puf_input:` chemin PUF Pagero input correspondant
- `puf_input_note:` explication + source citée en jaune
- `children:` TT enfants pour les TG (principe Laurent Malzac: TG = groupement automatique)
- Légende couleurs DGFiP XML: vert=TT (direct), jaune=TG (groupe dérivé), violet=TB (bloc racine)
- Filtres: scope AR/AP/AR+AP/PAIEMENT + flux 10/10.1/10.3/10.2-10.4 + recherche

### ILC — 20 statuts (200-214 + 224/225/226/227/228)
- Sous-vues: 🔄 Statuts ILC / 🚫 Codes Motif (intégré dans l'onglet ILC)
- `sides:` AR/AP/AR+AP/PA
- `pagero_code:` code Pagero (ex: PAYMENT_RECEIVED)
- `initiateur:` qui émet le statut
- `mdt218:` note MDT-218 v1.6 (211/212)
- `children:` TT enfants (TG)
- Nouveaux codes: 224 REQUEST_PAYMENT, 225 FACTORED, 226 FACTORED_CONFIDENTIAL, 227 ACCOUNT_CHANGED, 228 FACTORING_CANCELLED
- Panneau détail: Pagero RC + Reason codes + Amounts (MDT-218) + Action codes + Cas PUF (10 exemples AR)

### Consts Pagero
- `PAGERO_BP_CODES` (13 codes BT-23)
- `PAGERO_DT_CODES` (10 codes BT-3)
- `PAGERO_TC_CODES` (15 codes UNTDID 4451 BT-21)
- `PAGERO_SCHEMES` (14 Party Identification Schemes)
- `PAGERO_RESPONSE_CODES` (21)
- `PAGERO_REASON_CODES` (22)
- `PAGERO_AMOUNT_TYPES` (14)
- `PAGERO_ACTION_CODES` (7)
- `PAGERO_LC_FIELDS` (35 champs ApplicationResponse, incl. MDT-218 v1.6)
- `PUF_LC_EXAMPLES` (10 cas AR: Encaissée, Refus, Escompte, Retenue, Annulation MEN-, Troc, Affacturage, Sous-traitance, Co-traitance, Visée)

### BFR Rules — 100 règles Tradeshift
- Colonnes: # | Règle BFR | Catégorie | Champ BT AFNOR | Statut AFNOR
- Légende M/CM/O/— dans l'en-tête
- `cat:` catégorie (Vendeur FR, Acheteur, TVA, Ligne, Coty PO, Paiement...)
- `bt:` champ BT correspondant
- `afnor:` M/CM/O/- statut AFNOR

### Coty FR
- Cadres de facturation (B1/S1/M1 core, B2/S2/M2+B4/S4/M4 acomptes, B7/S7 VAT-on-payment)
- **Interco FR→FR** : HORS PÉRIMÈTRE — Tradeshift ignore VAT ID Coty/Coty (Edgar Palau 24/04/2026)
- **Règles SIREN** : régulier=standard / Média=SIREN/MEDIA / T&E=BT-58 discriminant

## Décisions projet confirmées (24/04/2026)
| Sujet | Décision | Source |
|-------|----------|--------|
| Interco FR→FR | Hors périmètre Tradeshift, pas de push SAP | Edgar Palau |
| SIREN fournisseurs réguliers | Schème 0002 standard | Edgar Palau |
| SIREN Média | Process Publicis spécifique | Edgar Palau |
| T&E discriminant | BT-58 email employé | Edgar Palau |
| Cadres de facturation | B1/S1/M1/B2/S2/M2/B4/S4/M4/B7/S7 — tous Yes | David |
| BT-23 dual-rôle | ProfileID=PA (technique) / @name=Coty (métier ZEIVTRCAT) | Laurent Malzac |
| TG = groupement auto | Activé dès que TT enfants présents | Laurent Malzac |

## Points normativement complexes intégrés
- BT-23: ProfileID (PA, technique) ≠ @name (Coty, cadre B1/S1/M1 → alimente TG-10/TT-28 DGFiP)
- TG vs TT: TG = groupement automatique, pas à renseigner directement
- MEN négatif (UC34): annulation Encaissée, code reste PAYMENT_RECEIVED
- Dual MEN barter (UC41): +12000@20% + -10000@0% = net 2000€
- Escompte (UC22a): MEN réduit + ESC, pas d'avoir pour services
- Retenue garantie (UC26): MEN 95% + RAP 5%
- Affacturage (UC10/225): distribution duale acheteur+factor
- Sous-traitance (UC13/224): BT-23=S5, distribution duale buyer+contractor
- Représentation fiscale: fournisseur étranger avec TVA FR → autoliquidation quand même

## Dernier commit
`ff651d9` — feat: Fix C (UC/PUF bug) + A (SYS enrichi + Y-Model SVG) + B (Calendrier + Recherche globale)

## Historique des commits récents
| Hash | Description |
|------|-------------|
| `ff651d9` | Fix C+A+B : UC bug, SYS enrichi, Y-Model SVG, Calendrier, Recherche globale |
| `3f2b5db` | docs: PROJECT_JOURNAL.md résumé complet |
| `68d3972` | feat: Coty FR — Interco FR→FR + règles SIREN |
| `8a7cb02` | feat: BFR Rules — légende colonnes |
| `4a0a2f`  | feat: BFR Rules — 100 règles mapping BT |
