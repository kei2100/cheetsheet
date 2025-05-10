# Kotlin

## Kotlin コードのマニュアルコンパイル

```bash
$ kotlinc <srcfile or directory> -include-runtime -d <jarname>
$ java -jar <jarname>  # 実行
```
## REPL

```bash
$ brew install ki
$ ki
```
## JVM のメモリ確認

```
Runtime.getRuntime().totalMemory()  // JVM が確保している現在のメモリ総容量
Runtime.getRuntime().freeMemory()  // JVM の空きメモリ
Runtime.getRuntime().maxMemory()  // JVM の最大メモリ。Xmx。
```
