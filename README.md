# Plateforme d'attestations — 100% gratuite

Générateur multi-événements/multi-templates + vérification en ligne par QR code.
Aucun coût : Supabase (base de données) et Netlify (hébergement) ont des paliers
gratuits largement suffisants pour ce type d'usage.

## Architecture

```
generator/          → génère les PNG (Python, tourne sur ton PC)
  templates/<nom>/   → un dossier par modèle d'attestation
  generate.py
  participants_exemple.csv

web/verify.html      → page de vérification (statique, à héberger sur Netlify)
supabase_setup.sql   → schéma de la base (à coller dans Supabase)
```

## Étape 1 — Créer le projet Supabase (gratuit)

1. Va sur https://supabase.com → crée un compte → "New project".
2. Une fois créé, ouvre **SQL Editor** → colle le contenu de `supabase_setup.sql` → exécute.
3. Dans **Project Settings > API**, note :
   - `Project URL` (ex: `https://xxxx.supabase.co`)
   - `anon public key`
   - `service_role key` (⚠️ secrète, jamais publiée sur un site)

## Étape 2 — Déployer la page de vérification (Netlify, gratuit)

1. Ouvre `web/verify.html`, remplace :
   - `TON-PROJET.supabase.co` → ton `Project URL`
   - `TA_CLE_ANON_PUBLIQUE` → ta clé `anon public` (celle-ci PEUT être publique, elle n'a que le droit de lecture grâce à la policy SQL).
2. Va sur https://app.netlify.com → glisse le dossier `web/` sur la page "Deploy" (drag & drop, aucune commande).
3. Netlify te donne une URL du type `https://ton-site.netlify.app`. Note-la.

## Étape 3 — Générer les attestations

```bash
cd generator
pip install -r requirements.txt

python generate.py participants_exemple.csv \
  --base-url "https://ton-site.netlify.app/verify.html?id=" \
  --supabase-url "https://xxxx.supabase.co" \
  --supabase-key "TA_CLE_SERVICE_ROLE"
```

⚠️ Pour générer (insérer dans la base), utilise la **service_role key**, jamais l'anon key
(l'anon key n'a que le droit de lecture, l'insertion échouera volontairement sinon).

Les PNG sortent dans `generator/output/`, prêts à distribuer.

## Ajouter un nouveau modèle d'attestation (mérite, organisateur, autre événement...)

1. Crée `generator/templates/mon_modele/template.png` (image de fond).
2. Crée `generator/templates/mon_modele/config.json` en copiant celui de `participation`
   et en ajustant les coordonnées (zones nom, QR, code) selon ton visuel.
3. Dans ton CSV, mets `mon_modele` dans la colonne `template`.

Aucune modification de code nécessaire — c'est le principe même de la plateforme :
un modèle = un dossier, réutilisable à l'infini pour tous tes événements futurs
(ESIG Tech Arena, THE CLASH, SAÉ, etc.).

## Vérifier une attestation

Scanner le QR (ou ouvrir `https://ton-site.netlify.app/verify.html?id=ETA-2026-0001`)
affiche "✓ Attestation authentique" avec les détails, ou "✕ Non trouvée" si le code
n'existe pas dans la base.
