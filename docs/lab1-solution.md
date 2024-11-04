# Lab 1 : Solution

##Explanation in English (french below)

![Solution](\assets\regex2.png){ align=left }

##Explanation in French

## Explication de l'expression régulière : `(?<=\bproxy\s)\d+`

This regular expression is designed to capture a number that appears immediately after the word "proxy " (with a space after it). Here is a detailed explanation of each component of the expression:

### Expression Details

- `(?<= ... )` - **Positive Lookbehind Assertion**

  `(?<= ... )` is a **positive lookbehind assertion**. A lookbehind is a type of assertion that checks if a certain pattern appears before the current position in the text string. An important aspect of lookbehind is that it does not include the text it checks in the final match result.

  In this case, `(?<=\bproxy\s)` ensures that "proxy" followed by a space appears immediately before the position where the capture begins.

- `\b` - **Word Boundary**

  `\b` is a **word boundary**, which matches the position between a word character (e.g., `[A-Za-z0-9_]`) and a non-word character. Here, `\b` before "proxy" ensures that "proxy" is matched as a whole word. For example, it would not match "xproxy" or "proxying" because these words do not fully match "proxy".

- `proxy` - **Literal Text**

  This part of the expression matches the exact word "proxy". Combined with `\b` and `\s`, it ensures that only numbers appearing after a complete match of the word "proxy " (with a space after it) are captured.

- `\s` - **Whitespace Character**

  `\s` matches any whitespace character (such as a space, tab, etc.). Here, it ensures that there is at least one space after "proxy" before the number begins.

- `\d+` - **Digit(s)**

  `\d` matches a single digit (0-9). The quantifier `+` means "one or more" of the preceding character (in this case, `\d`). Together, `\d+` matches one or more consecutive digits, representing the number (such as a port number) that follows "proxy ".

---

### How the Complete Expression Works

**Complete Expression**: `(?<=\bproxy\s)\d+`

1. The **lookbehind** `(?<=\bproxy\s)` ensures that the expression finds a sequence of digits following the word "proxy" as an isolated word, immediately followed by a space.
2. The `\d+` part captures the number (one or more digits) that appears after "proxy ".

---

### Usage Example

To illustrate, if we apply this expression to the following string:

```plaintext
proxy 8080
```

The pattern `(?<=\bproxy\s)\d+` will capture `8080` because:

- "proxy" is followed by a space,
- `8080` is a set of consecutive digits that appears right after "proxy ".

![Solution](\assets\regex2.png){ align=left }

Cette expression régulière est conçue pour capturer un nombre qui apparaît immédiatement après le mot "proxy " (avec un espace après). Voici une explication détaillée de chaque composant de l'expression :

### Détails de l'expression

- `(?<= ... )` - **Assertion de Lookbehind positif**

  `(?<= ... )` est une **assertion de lookbehind positif**. Un lookbehind est un type d'assertion qui vérifie si un certain motif apparaît avant la position actuelle dans la chaîne de caractères. Ce qui est important avec un lookbehind, c'est qu'il ne capture pas le texte qu'il vérifie dans le résultat final de la correspondance.

  Dans ce cas, `(?<=\bproxy\s)` vérifie que "proxy" suivi d'un espace apparaît immédiatement avant la position où l'on souhaite commencer la capture.

- `\b` - **Délimiteur de mot (Frontière)**

  `\b` est un **délimiteur de mot**, qui correspond à la position entre un caractère de mot (par exemple, `[A-Za-z0-9_]`) et un caractère non-mot. Ici, `\b` avant "proxy" s'assure que "proxy" est capturé comme un mot complet. Par exemple, cela ne correspondra pas à "xproxy" ou "proxying", car ces mots ne correspondent pas entièrement à "proxy".

- `proxy` - **Texte littéral**

  Cette partie de l'expression correspond exactement au mot "proxy". Combiné avec `\b` et `\s`, cela garantit que seuls les nombres situés après une correspondance complète du mot "proxy " (avec un espace après) sont capturés.

- `\s` - **Caractère d'espace**

  `\s` correspond à n'importe quel caractère d'espace (comme un espace, une tabulation, etc.). Ici, il est utilisé pour s'assurer qu'il y a au moins un espace après "proxy" avant que le nombre commence.

- `\d+` - **Chiffre(s)**

  `\d` correspond à un chiffre unique (0-9). Le quantificateur `+` signifie "un ou plusieurs" du caractère précédent (ici, `\d`). Ensemble, `\d+` capture un ou plusieurs chiffres consécutifs, représentant le nombre (comme un numéro de port) qui suit "proxy ".

---

### Comment fonctionne l’expression complète

**Expression complète** : `(?<=\bproxy\s)\d+`

1. Le **lookbehind** `(?<=\bproxy\s)` s'assure que l'expression trouve une séquence de chiffres qui suit le mot "proxy" en tant que mot isolé, suivi immédiatement par un espace.
2. La partie `\d+` capture le nombre (un ou plusieurs chiffres) qui vient après "proxy ".

---

### Exemple d'utilisation

Pour illustrer, si nous appliquons cette expression à la chaîne suivante :

```plaintext
proxy 8080
```

Le motif ```plaintext (?<=\bproxy\s)\d+``` capturera ```plaintext 8080``` parce que :

- "proxy" est suivi d'un espace,
- `8080` est un ensemble de chiffres consécutifs, juste après "proxy ".

##Explanation in English