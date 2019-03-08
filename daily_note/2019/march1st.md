# Done
- paiza * 2
- CheckIO * 1
- LinuxコマンドでログデータからユニークユーザーIDをカウントする

# Learn
## Linux
- **tail** : by -n option, we can see file from any line
- **cut** : by -f option, we can extract the column from file(in addition, -d is used to specify dilimiter)
- **uniq** : by -c option, we can remove dupulicates and count the occurrences of each uniq line(for this command, data must be sorted orderly)
- **wc** : by -l option, we can count the number of line of file
- **grep** : by -v option, we can search strings which don't match the pattern
## Python
- 組み込み関数divmod：引数a, bを渡すことで(a // b, a % b)が返ってくる
- datetimeモジュール
  - 時間を扱うモジュール
  - 実際に時間差を計算する上ではdatetimeオブジェクトまたはdateオブジェクトを使い、そのアウトプットがtimedeltaオブジェクトになる
  - timedeltaオブジェクトとintオブジェクトは演算が不可能
  - timedeltaそのものはあくまでも時間と時間の差なので、ナマで扱うのは使い勝手が良くない
  - 最初っからtimedeltaオブジェクトを生成するのではなく、datetime, dateオブジェクトなどをテキトーな日付で生成して計算を行い、%H, %Mなどでほしい属性にaccessすればOK
  - また、datetime.time型はdatetime型に変換しないと計算ができない

