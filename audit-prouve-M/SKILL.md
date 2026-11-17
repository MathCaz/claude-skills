---
name: audit-prouve-M
description: Auditer un document long et chiffré, appliquer les corrections, puis PROUVER mécaniquement que chacune est passée. Use when auditing, fact-checking or correcting a large document (HTML/Markdown report, financial model, analysis) where numbers must agree across sections — and whenever you need to be sure nothing was half-corrected or silently skipped. Triggers: "audite ce document", "vérifie qu'il ne reste rien", "relis et corrige", "prouve que c'est appliqué", "balayage".
argument-hint: "[fichier] [parties à auditer]"
---

# Auditer, corriger, prouver

Trois phases, dans cet ordre. **La troisième n'est pas optionnelle** : sans elle on ne sait pas si
une correction est passée, et on redécouvre les mêmes restes à chaque session.

## Phase 1 — Auditer sur un instantané figé

1. **Figer** une copie du document (`snapshot-<date>-<lot>.ext`). Les auditeurs lisent la copie,
   jamais le fichier vivant.
2. **Paralléliser par partie**, pas par type d'erreur : un agent par chapitre ou par lot de
   sections. Proposer le découpage et son coût avant de lancer.
3. **Un seul écrivain** — toi. Les agents sont en lecture seule et rendent un fichier de rapport
   chacun dans le scratchpad. Les contradictions entre agents se tranchent **à la source**, jamais
   en moyennant deux avis.
4. **Dire explicitement quelles parties n'ont jamais été auditées.** « Personne n'a vérifié » n'est
   pas « il n'y a rien ».

### Le contrat de rapport — non négociable

Chaque rapport finit par un tableau aux colonnes **exactement** celles-ci :

```
| id | ligne | section | gravité | défaut | valeur affichée | valeur juste |
```

**`valeur affichée` contient la chaîne fautive telle qu'elle est dans le fichier** — le nombre avec
ses espaces, ses entités HTML, sa ponctuation. C'est la clé de toute la phase 3. Un rapport sans
cette colonne est à refaire.

## Phase 2 — Corriger avec un harnais qui échoue avant d'écrire

Toujours un script Python jetable dans le scratchpad, exécuté avec `python3 -I`, **jamais** `sed`
sur un fichier structuré.

```python
import re, io
s = io.open(P, encoding='utf-8').read()

def rep(o, n, lbl, k=1):
    global s
    c = s.count(o)
    assert c == k, lbl + ' : ' + str(c) + ' au lieu de ' + str(k)
    s = s.replace(o, n)

# ... toutes les corrections ...
# ... puis TOUS les contrôles ...
io.open(P, 'w', encoding='utf-8').write(s)   # l'écriture vient en dernier
```

**L'ordre compte** : les `assert` passent avant le `write`, donc un ancrage raté laisse le fichier
intact plutôt qu'à demi patché.

Contrôles de structure à rejouer après chaque passe sur du HTML :

```python
bad = [t for t in ['table','div','section','tbody','thead','tr','ul','li','p','td','th',
                   'strong','span','small','em','ol','h2','h3','h4','sup','b']
       if len(re.findall(r'<'+t+r'[ >]', s)) != len(re.findall(r'</'+t+r'>', s))]
assert not bad, bad
```

Et, si le document a des renvois internes numérotés, vérifier qu'aucun ne pointe dans le vide.

### Pièges de manipulation

- **Encodage mixte** : un même fichier peut mêler UTF-8 brut et entités HTML selon les sections.
  Toujours dumper les octets exacts (`re.finditer` + slicing) avant d'écrire une ancre.
- **`%` dans les chaînes** casse le formatage `%s` — concaténer plutôt que formater.
- **Ancrage de contrôle** : vérifier qu'on parse bien le tableau visé en l'ancrant sur **deux**
  valeurs qui n'existent ensemble que là.

## Phase 3 — Prouver

Rejouer **toutes** les colonnes `valeur affichée` des rapports contre le fichier corrigé :

```python
pat = re.compile(r'^\|\s*([A-Z]\d+[a-z]?)\s*\|[^|]*\|[^|]*\|[^|]*\|[^|]*\|\s*`([^`]+)`\s*\|', re.M)
for f in sorted(glob.glob(SCR + 'audit*.md')):
    for ident, val in pat.findall(io.open(f, encoding='utf-8').read()):
        if len(val.strip()) >= 4 and val.strip() in s:
            print('RESTE ' + ident + ' >>> ' + val.strip())
```

**Chaque chaîne encore présente s'inspecte à la main, une par une, en contexte.** Il y a quatre
issues légitimes, et il faut nommer laquelle s'applique :

1. **artefact de sous-chaîne** — « 7 000 » trouvé dans « 27 000 » ;
2. **correction par ajout** — la chaîne d'origine reste normalement en place, un complément a été
   ajouté à côté ;
3. **montant conservé à dessein**, avec sa convention déclarée dans le document ;
4. **mention historique volontaire** — l'ancienne valeur citée pour expliquer pourquoi elle a changé.

Tout le reste est un défaut non appliqué.

## La classe d'erreurs que ce protocole existe pour attraper

**Les demi-corrections** : la valeur est corrigée dans le tableau et laissée dans la prose qui le
commente. Elles sont invisibles à la relecture parce qu'on vérifie que *la bonne valeur est
arrivée*, jamais que *l'ancienne est partie*. Vérifier les deux.

## Corriger ou déclarer

Quand un chiffre est fragile mais largement propagé, **déclarer l'hypothèse et sa sensibilité** vaut
mieux que propager vingt cellules. Corriger quand la valeur est fausse et peu diffusée ; déclarer
quand la propagation est large et l'enjeu faible. Dans les deux cas, l'écrire dans le document —
pas seulement dans le rapport.

**Ne jamais annoncer qu'il ne reste rien sans avoir passé la phase 3.**
