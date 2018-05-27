# for文におけるcontinue文の使用について
for文を書く時の、continue使用について議論になったので、考えてみました。

for文の中でif文を書き、特に条件が複雑になるとすぐにネストが深くなってしまいます。

そこでcontinueを使用すると、ネストを浅くすることができるケースがありますが、無闇やたらに使用するのも間違いだと思います。

以下、どのような時に使用するのがいいかコード例とともに書いてみます。


### コード例
```java
// コード1
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
        if (B != null) {
            method("AB");
            if (C != null) {
              method("ABC");
            }
        }
    }
}
```

コード1は、条件を素直に書いたパターンでありネストが深めになっています。

このコードをcontinueを使用せずにネスト浅くする書き方をしてみます。
```java
// コード2
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
    }
    if (A != null && B != null) {
        method("AB");
    }
    if (A != null && B != null && C != null) {
        method("ABC");
    }
}
```

コード2では、あえて条件を重複させて書くことによりコード1より行数は少なくなっていますがネストを浅くすることができています。

最後にこれをcontinueを使用して書き直してみると、
```java
// コード3
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A == null) {
        continue;
    }
    method("A");
    if (B == null) {
        continue;
    }
    method("AB");
    if (C == null) {
        continue;
    }
    method("ABC");
}
```
のようになります。

ただし今回の例の前提として、呼び出すmethodはオブジェクトA、B、Cを一切操作しないものとしてください。

（この前提が崩れるとコード3は他の二つとは別の処理をするものとなってしまいます。）

今回、同じ処理を3通りの書き方で書いてみましたが、それぞれの書き方を以下のように命名してみたいと思います。

* コード1: 条件重複なしパターン
* コード2: 条件重複ありパターン
* コード3: continueパターン


### 今回のケースでは、どれが一番わかりやすい？
多分可読性的に一番わかりやすいのは条件重複ありパターンであるコード2だと思います。

continueパターンであるコード3は、コード2と同様にネストは取り除くことができていますが、if文+continueの処理とmethodの実行が交互に来ていて気持ちが悪いです。

またif文とmethodの関連性がいまいちという点でも可読性悪いと思います。

（コード2のようにifの条件を満たすとき、methodが実行されるの方が明らかにわかりやすいです。）


### 条件重複ありパターンで問題が出てくるとき
例えば今回のコード1に追加の処理を書くことになったとします。
```java
// コード4
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
        if (C == null) {
            method("A!C");
        }
        if (B != null) {
            method("AB");
            if (C != null) {
              method("ABC");
            }
        } else {
            method("A!B");
        }
    }
}
```

どんどん可読性悪くなって来ました。

これを条件重複ありパターンの書き方にすると次のようになります。
```java
// コード5
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
    }
    if (A != null && C != null) {
        method("A!C");
    }
    if (A != null && B != null) {
        method("AB");
    }
    if (A != null && B != null && C != null) {
        method("ABC");
    }
    if (A != null && B == null) {
        method("A!B");
    }
}
```

こうやって書いて行くとわかりますが、条件重複ありパターンだとmethodを増やすたびに実行に必要な条件を全部記述しなければならないことになります。

変にコードの記述量が増えるのは、書き間違いによるバグの原因にもなりやすいかなと思います。

じゃあ、少しネスト増やして次のように書こうかという話になってくるかもしれません。
```java
// コード6
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
        if (C != null) {
            method("A!C");
        }
        if (B != null) {
            method("AB");
        }
        if (B != null && C != null) {
            method("ABC");
        }
        if (B == null) {
            method("A!B");
        }
    }
}
```

元のコードよりは条件文の記述量は減りましたが、その分ネストが少し深くなってしまいました。

今後、さらに処理や変数が増えればもっとネストが深くなってしまう可能性もあります。


### continueの導入
今回のコード6を再度見てみると、どのmethodが実行される時もAがnullでないことが条件として求められています。

このような場合であれば、continueを使用するとネストを浅くでき可読性もいいケースがあります。

```java
// コード7
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A == null) {
        continue;
    }

    method("A");
    if (C != null) {
        method("A!C");
    }
    if (B != null) {
        method("AB");
    }
    if (B != null && C != null) {
        method("ABC");
    }
    if (B == null) {
        method("A!B");
    }
}
```
この導入の仕方は、早期リターンと基本的に考え方は同じです。

今回の `method("A");` 以下のコードはAがnullでないことが前提の時に実行される処理です。

そのため、Aがnullの時は実行される必要がないのでcontinueしています。

つまり、Aがnullの時というのは例外的なケースということになります。

そのため
```java
if (A == null) {
    continue;
}
```

は例外処理であり、メインの処理はそれ以降に記述していることになります。

このように考えると、メインの処理と例外処理をうまく前半と後半に分割した書き方になっているとも言えます。


### A == null条件の処理を追加する時は？
ここまで見て来たケースだと、Aがnullでない時の処理しかないからこそcontinueを導入できたとも言えます。

そしてそれは実際に正しいと思います。

それでは、Aがnullの時の条件が入って来た時にcontinueを導入するとどうなるかをアンチパターンとして書いて見たいと思います。

まず元コードを重複なしパターンで書いてみます。
```java
// コード8
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
        if (B != null) {
            method("AB");
            if (C != null) {
                method("ABC");
            }
        }
    } else {
        method("!A");
        if (B != null) {
            method("!AB");
            if (C == null) {
                method("!AB!C");
            }
        } 
    }
}
```

これを重複ありパターンで書くと次になります。
```java
// コード9
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
    }
    if (A != null && B != null) {
        method("AB");
    }
    if (A != null && B != null && C != null) {
        method("ABC");
    }

    if (A == null) {
        method("!A");
    }
    if (A == null && B != null) {
        method("!AB");
    }
    if (A == null && B != null && C == null) {
        method("!AB!C");
    }
}
```

そしてコード9をcontinueパターンで書いてみます。

今回はAがnullの条件だった時に、最後にcontinueする形で書いてみます。
```java
// コード10
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
        if (B != null) {
            method("AB");
        }
        if (B != null && C != null) {
            method("ABC");
        }

        continue;
    }

    method("!A");
    if (B != null) {
        method("!AB");
    }
    if (B != null && C == null) {
        method("!AB!C");
    }
}
```

今回この書き方が良いかどうかは、 `if (A != null)` のスコープでcontinueされるまでの処理の意味合いによって変わってくると考えています。

ここの処理が、Aが例外的な状態であるため、その後始末をするような処理であればこの書き方でも問題はないと思います。

（そのようなパターンでもメインの処理と同様のコード量だと誤解を与える可能性も高いですが。）


しかし、AがnullであることとAがnullでないことは同列な関係で単なる分岐であるとすると、それぞれの処理が記述されるインデントが異なるため、同列であることが伝わりにくいです。

そうであれば、continueを使用せずに素直にelseを使用した方が可読性がいいでしょう。
```java
// コード11
for (Sample sample: sampleList) {
    Object A = sample.getA();
    Object B = sample.getB();
    Object C = sample.getC();

    if (A != null) {
        method("A");
        if (B != null) {
            method("AB");
        }
        if (B != null && C != null) {
            method("ABC");
        }
    } else {
        method("!A");
        if (B != null) {
            method("!AB");
        }
        if (B != null && C == null) {
            method("!AB!C");
        }
    }
}
```

コード11が見やすいコードかどうかというと微妙ですが、コード10のように無闇にcontinueを使用するよりはマシです。


### まとめ
結局、continueを使用する時の考え方は早期リターンと同様にするべきだと思います。

つまり、 **その条件が例外的なものである時** です。

実際にそのような条件はたくさんあるのではないのかと思います。

あるListから特定の型の要素だけ抽出して、その要素に対してのみ処理をするだとか。

（この場合、その特定の型以外の要素は例外的なものとして、一切何もしないケースを想定しています。）

ただ、条件自体少ないならcontinue使用しないで書く方が短くかけて可読性もいい時も当然あります。

その辺はバランス見ながら、書き方選んでいくのが大切かなと思います。

