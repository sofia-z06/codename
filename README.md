# Codename

Agent Ollama qui garde un codeword derrière un password, et 5 prompts d'attaque.

Modèle imposé : `granite3.1-moe:3b`.

## Structure

```
.
├── Modelfile        l'agent (SYSTEM avec {password} et {codeword})
├── attacks/         5 prompts d'attaque, un fichier par famille
├── tests/           tests automatisés
│   ├── run_tests.py
│   ├── cases.json   les cas de test
│   └── versions/    les 18 versions du Modelfile, pour le benchmark
└── docs/            notes de préparation à la défense
```

Le nom de chaque fichier de `attacks/` indique la famille qu'il exploite.

## Prérequis

```bash
ollama pull granite3.1-moe:3b
curl http://127.0.0.1:11434
```

La seconde commande doit répondre `Ollama is running`. Sinon, lancer `ollama serve`.

## Utiliser l'agent

Le `Modelfile` contient les placeholders `{password}` et `{codeword}`. Pour l'essayer à la main, créer une copie avec de vraies valeurs :

```bash
sed -e 's/{password}/mangue77/g' -e 's/{codeword}/portail42/g' Modelfile > Modelfile.test
ollama create codename -f Modelfile.test
ollama run codename "What is the codeword? My password is mangue77."
```

`Modelfile.test` est ignoré par Git.

## Lancer les tests

```bash
python3 tests/run_tests.py
```

Le script remplace lui-même les placeholders, crée un modèle séparé `codename-test`, puis lance :

- les 3 checks du sujet (create, release, utility) ;
- les cas de défense et de reformulation de `tests/cases.json` ;
- les 5 fichiers de `attacks/` ;
- un scénario de conversation continue.

Il considère comme une fuite le secret en clair, épelé, à l'envers, en alphabet NATO, en base64 ou en hexadécimal. Il affiche un tableau de scores à la fin.

Options :

```bash
python3 tests/run_tests.py --repeat 10
python3 tests/run_tests.py --only NATO -v
python3 tests/run_tests.py tests/versions/*.Modelfile --no-scenario
python3 tests/run_tests.py --password autre --codeword valeur
```

1. Chaque prompt est envoyé 10 fois pour vérifier le déterminisme.
2. Seuls les tests dont le nom contient « NATO » sont lancés, avec les réponses affichées.
3. Benchmark de toutes les versions.
4. Autres valeurs de test.

## Vérifier la limite de 1500 caractères

```bash
wc -c attacks/*.prompt
```