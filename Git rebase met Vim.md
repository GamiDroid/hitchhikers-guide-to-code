#git #rebase #vim

Tijdens het ontwikkelen is het soms handig om zo veel mogelijk commits te maken. Het kan natuurlijk soms voorkomen dat je iets vergeten te wijzigen bent en dat dit toch snel mee genomen moet worden in de eerdere commit.

Hiervoor kan het `git rebase` voor gebruikt worden. Deze kan commits samenvoegen maar ook de volgorde veranderen.
Op deze pagina wordt beschrijven hoe deze acties uitgevoerd kunnen worden.

Voer een rebase uit op de laatste 5 commits van de huidige branch.

```bash
git rebase -i HEAD~5
```

Resultaat:
```
pick e16b7b8 Implement axaptaxml helper in apis
pick 8b72978 Remove unused axapta models
pick 50e5ea6 Add axapta xml caching class
pick 068bc13 Refresh cache when any of the requested route id caches is empty
pick 144d01c Change scope helper method
```

Het resultaat dat ik wil bereiken is dat commit `144d01c` in de commit `e16b7b8` erbij komt.

Wat hiervoor moet gebeuren is dat commit `144d01c` eerst onder commit `e16b78` gezet moet worden. Vervolgens willen we dat deze dan wordt ge-squashed in de andere commit.

Ga in `NORMAL` mode op de lijn van `144d01c` staan en gebruik vervolgens het commando `dd` om de geselecteerde lijn te knippen.

Vervolgens kan door met de cursor op lijn `e16b7b8` te staan commando `p` gebruikt worden om de lijn te plakken.

Resultaat:
```
pick e16b7b8 Implement axaptaxml helper in apis
pick 144d01c Change scope helper method
pick 8b72978 Remove unused axapta models
pick 50e5ea6 Add axapta xml caching class
pick 068bc13 Refresh cache when any of the requested route id caches is empty
```

Door nu de cursor voor de `pick` van `144d01c` te zetten kan het commando `de` (delete to end word) gebruikt worden om het woord te verwijderen. Vervolgens wordt het command `i` gebruikt om in `INSERT` mode te komen. Type nu `s` of `squash` om de commit toe te voegen aan de bovenstaande commit.

Resultaat:
```
pick e16b7b8 Implement axaptaxml helper in apis
squash 144d01c Change scope helper method
pick 8b72978 Remove unused axapta models
pick 50e5ea6 Add axapta xml caching class
pick 068bc13 Refresh cache when any of the requested route id caches is empty
```

Druk nu `ESC` om naar `NORMAL` mode terug te keren. En gebruik `:wq` om de wijzigingen op te slaan en [[Vim]] te sluiten.
# Handige Vim Tips

`dd` : delete (cut) a line 
`yy` yank (copy) a line
`p`: put (paste) the clipboard after cursor

[Vim cheatsheet](https://vim.rtorr.com/)

