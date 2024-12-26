#git #git-sparse-checkout #sparse-checkout

In git is het mogelijk om specifieke folders te klonen, doormiddel van `sparse-checkout`. Dit is handig omdat dan niet de gehele repository op de machine gedownload wordt, dit scheelt geheugen.

Het kan ook handig zijn, om grotere repositories hierdoor op te splitsen. Zodat wijzigingen van een andere project niet per ongelijk ingecheckt worden.
# References
https://git-scm.com/docs/git-sparse-checkout

# Example

```bash
git sparse-checkout init
# same as:
# git config core.sparseCheckout true

git sparse-checkout set "A/B"
# same as:
# echo "A/B" >> .git/info/sparse-checkout

git sparse-checkout list
# same as:
# cat .git/info/sparse-checkout
```
