# Emacs
> An extensible, customizable, free/libre text editor  
[GNU Emacs - GNU Project](https://www.gnu.org/software/emacs/)

## 同じ処理を繰り返し実行するキーボードマクロ
1. `C-x (` でキーボードマクロの記録を開始
2. キーボード操作
3. `C-x )` でキーボードマクロの記録を終了
4. `C-xe` で実行  

複数回実行したい時は、 `C-xe` の前に `C-u 回数` をつける  
```bash
C-u 100 C-xe
```
