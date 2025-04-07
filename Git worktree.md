#git #worktree

> git stash vervanger

Git worktree is een commando die gebruikt kan worden om gemakkelijk een extra instantie te maken van de lokale git repo. Dit kan handig zijn, zodat je niet constant stashes hoeft aan te maken.

# Worktree aanmaken

```cmd
git worktree add ..\<new_instance_folder> <branch_name>
```

voorbeeld: 

Op dit moment ben ik aan het werk in de branch 'working'. Maar zou graag tussendoor een hotfix willen uitvoeren.

```cmd
repos\hitchhikers-guide-to-code> git branch
  main
* working
```

Ik maak een worktree aan voor de 'main' branch. Deze wordt in de folder 'hitchhikers-guide-to-code-hotfix' gezet.

```cmd
git worktree add ..\hitchhikers-guide-to-code-hotfix main
```

> Ik raad aan geen worktree aan te maken, als sub folder van de hoofd worktree. Je kan namelijk per ongelijk de bestanden twee keer toevoegen in de repo.

```cmd
repos\hitchhikers-guide-to-code> ls ..
hitchhikers-guide-to-code
hitchhikers-guide-to-code-hotfix
```

Nu kan ik in de extra folder werken alsof aan de hotfix en blijft de oude 'working' branch nog actief.

In de nieuwe worktree kan ik niet naar de 'working' branch switchen, omdat er al een worktree actief is om deze branch.

# worktree list

Doormiddel van het volgende commando kan ik een lijst krijgen van alle actieve worktree's met de locatie en de actieve branch.

```cmd
git worktree list
repos\hitchhikers-guide-to-code         adb7aae [working]
repos\hitchhikers-guide-to-code-hotfix  adb7aae [main]
```

# Worktree verwijderen

```cmd
git worktree remove ..\hitchhikers-guide-to-code-hotfix
```

Dit commando verwijderd de worktree (als er geen wijzigingen openstaan).
# Referenties

[Youtube - Philomatics - I was wrong about git stash...](https://www.youtube.com/watch?v=ntM7utSjeVU&list=WL&index=65)

