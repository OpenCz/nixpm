# nixpm - Points a upgrader

## Critique

1. Commandes promises mais non implementees
- Le script affiche `list` et `search` dans l'aide, mais ne route que `install` et `remove`.
- `remove` est un placeholder (`print(...)`) et ne modifie pas la config.
- Impact: l'interface publique est trompeuse et certaines commandes cassent le workflow attendu.

2. Incoherence forte entre code et README
- Le README documente `rm`, `ls`, `s` et `--no-rebuild`, mais aucun de ces alias/options n'est implemente.
- Le README annonce un backup `.bak` et une confirmation, absents du script.
- Impact: utilisateur en erreur, perte de confiance, risque de mauvaises manips.

3. Detection d'installation incorrecte
- `cmd_install` utilise `if package in content`, ce qui fait des faux positifs (ex: `git` trouve `git-lfs`).
- Impact: paquets non installes alors que l'outil dit le contraire.

## Eleve

4. Parsing fragile de `environment.systemPackages`
- Recherche par string exacte: `'environment.systemPackages = with pkgs; ['`.
- Ne gere pas les variantes de formatting, commentaires, multi-lignes, ou autres structures Nix.
- Impact: echec silencieux (`False`) ou insertion au mauvais endroit.

5. Gestion d'erreur insuffisante
- `read_text()` / `write_text()` sans `try/except` ni message detaille.
- Si le bloc package n'est pas trouve, `write_package_to_config` renvoie `False` mais `cmd_install` n'agit pas sur ce retour.
- Impact: fichier potentiellement inchange avec rebuild lance quand meme.

6. Rebuild lance meme en cas d'entree invalide
- `nixpm install` sans package lance quand meme `nixos-rebuild switch`.
- Impact: operation inutile, lente et potentiellement risquee.

## Moyen

7. Chemin de config potentiellement non standard
- `CONFIG_PATH` pointe vers `/etc/nixos/config.nix`.
- Sur beaucoup d'installations NixOS, le fichier standard est `/etc/nixos/configuration.nix`.
- Proposition: auto-detection (`configuration.nix`, puis fallback), ou option CLI `--config`.

8. Format d'ecriture et robustesse IO
- Pas d'ecriture atomique (temp file + rename), pas de lock, pas de backup.
- Impact: risque de corruption sur interruption.

9. UX CLI minimale
- Pas de codes retour coherents dans certains chemins d'erreur.
- Message `Invalid function` peu explicite (proposer la liste des commandes valides).

## Quick wins (ordre recommande)

1. Implémenter `argparse` avec:
- sous-commandes: `install`, `remove`, `list`, `search`
- alias: `rm`, `ls`, `s`
- options: `--config`, `--no-rebuild`, `--yes`

2. Corriger l'analyse des paquets:
- parser precis du bloc `environment.systemPackages`
- comparaison par tokens exacts, pas par substring.

3. Ajouter securite des modifications:
- backup `.bak`
- ecriture atomique
- validation du diff avant rebuild

4. Ne rebuild que si changement effectif
- si aucun changement: sortie propre sans `nixos-rebuild`.

5. Aligner README et implementation
- retirer les features non implementees, ou les implementer avant publication.

## Suggestions de refacto

- Decouper en fonctions pures:
  - `read_config(path)`
  - `extract_system_packages(text)`
  - `add_packages(text, packages)`
  - `remove_packages(text, packages)`
  - `write_config(path, text, backup=True)`
- Ajouter une couche `main()` uniquement pour la CLI.
- Ajouter tests unitaires sur des exemples de blocs Nix (formats differents).

## Mini checklist avant release

- [ ] `install` fonctionne avec 1+ paquets
- [ ] `remove` retire reellement les paquets
- [ ] `list` affiche les paquets detectes
- [ ] `search` execute une recherche nixpkgs utile
- [ ] `--no-rebuild` fonctionne
- [ ] backup + ecriture atomique actives
- [ ] README conforme au comportement reel