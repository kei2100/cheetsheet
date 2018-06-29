## 個人的にignoreしたい

`.git/info/exclude`に書き込む


## merge commitのrevert

```
git show <マージコミットのhash> する

commit 8ae52de41110e4a27c765f6b68f5a3c19b0dd199
Merge: 0e2695a47 2082dfba2

0e2695a47がマージコミット（親）ならば、「-m 1」
2082dfba2が親ならば「-m 2」として、

git revert 0e2695a47 -m 1
のようにする
```
