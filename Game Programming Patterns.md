# Game Programming Patterns

## 1章　アーキテクチャ、実行速度、ゲーム

### 1.1

- 良いコードとは「部分的な変更があっても全体に影響を及ぼさない」コード
  - すなわち、変更が容易なコード

- 良いアーキテクチャとは、コードの変更に必要な知識を最小にするもの
  - すなわち、しっかり分離が行われているアーキテクチャ

### 1.2

- コンポーネントの分離にもコストがかかる
- また、分離を頑張りすぎると抽象レイヤーやインターフェースが多すぎて意味わからんくなる

### 1.3

- コードの柔軟性を上げると、実行速度が遅くなる
- しかし、柔軟なコードを早くするほうが、早いコードを柔軟にするより簡単なので、最初は柔軟に書いて、終盤にチューニングする方が良い

### 1.4

- アーキテクチャを考慮するのにもコストがかかるため、プロトタイピングではそこまでやる必要はない
  - ただし、そのコードがすぐに捨てられる事が条件
  - 本番のコードとして使いまわしてはいけない

### 1.6

- コードをきれいに保つ方法はコードを単純に保つこと

## 2章 コマンド

- コマンドとは、「メソッド呼び出しを具現化したもの」である
  - 具現化とは、クラスなどのオブジェクトにして変数に代入したり、関数の引数に渡したりできるようにすること

### 2.3
  
- Undo機能の実装について
  - Commandクラスに、excute()とundo()を実装することで、「処理を巻き戻す」が簡単に実装できるようになる

## 3章 フライウェイト

- GPUの描画負荷軽減について

## 4章 オブザーバ

- Rx系の使うパターン
- 発信者側は誰がどんな目的で使うか知らんけどとりあえず発信し続けてる
- オブザーバパターンを使うのにふさわしくない場面について
  - 受信者と送信者の両側を同時に考える必要がある場合は使うべきではない
    - 「物理演算」と「実績解除システム」のように、両者にほとんど関連がない場合は使っても良い
    - ModelとView(Actor)も同様か

## 5章 プロトタイプ

- Monsterクラスを継承したGhost、Demon、Sorcererクラスを作成する事を考える
- この時に、各クラスのFactoryクラス(GhostFactory、DemonFactory...)をそれぞれ作るのはアホらしい
- そのため、MonsterクラスにClone()メソッドを実装し、FactoryクラスはコンストラクタでプレハブとなるMonsterインスタンスを渡してもらう
  - インスタンスを生成する時は、渡されたプレハブのClone()を呼び出す
  - 適切なプレハブを用意すれば、同じクラスでも様々なタイプのオブジェクトを生成できる(素早いゴースト、タフなゴーストなど)

```c#
public abstract Monster{
  public abstract Monster Clone();
}

public class Ghost : Monster{
  private int _health;
  private float _speed;

  public Ghost(int health,float speed){
    _health = health;
    _speed = speed;
  }

  public override Monster Clone(){
    return new Ghost(_health,_speed);
  }
}

public class MonsterFactory{
  private Monster _prefab;
  public MonsterFactory(Monster prefab){
    _prefab = prefab;
  }

  public Monster Create(){
    return _prefab.Clone();
  }
}
```

- しかし、この方法にも欠点が多い
  - 各クラスにClone()を実装するのでは、結局コード量が変わっていない
  - 浅いコピーと深いコピーのどっちをするの問題
    - 例えば、Ghostクラスが槍を持っていた場合、それは複製するの、しないの

- 別のFactoryクラスとして、ジェネリッククラスを使う手法もある
  - ただ、この方法だとGhostのコンストラクタに値を代入できなくない？

```c#
public abstract Monster{
}

public class Ghost : Monster{
  public Ghost(){
  }
}

public class Factory<T> where T : Monster{
  public Monster Create(){
    return new T();
  }
}
```

- 関数ポインタを使う方法もある
  
```c#
public abstract Monster{
}

public class Ghost : Monster{
  private int _health;
  private float _speed;

  public Ghost(int health,float speed){
    _health = health;
    _speed = speed;
  }
}

public class Factory{
  private Func<Monster> _spawnFunc;

  public Factory(Func<Monster> spawnFunc){
    _spawnFunc = spawnFunc;
  }

  public Monster Create(){
    return _spawnFunc?.Invoke();
  }
}

public class User{
  private Factory _fasterGhostFactory;
  private Factory _toughGhostFactory;
  public User(){
   _fastGhostFactory = new Factory(()=>new Ghost(15,30f));
   _toughGhostFactory = new Factory(()=>new Ghost(50,5f));
  }

  private void Hoge(){
    //ゴーストをそれぞれ10体ずつ作成
    for(int i=0;i<10;i++){
      var fasterGhost = _fasterGhostFactory.Create();
      var toughGhost = _toughGhostFactory.Create();
    }
  }
}
```

- いずれの場合においても、これだけだと何の意味もない
  - Factoryクラス内で共通の作業(ゲームループへの登録など)を行ってこそ真価を発揮する

## 6章 シングルトン

- シングルトンはクラスによりカプセル化されたグローバル変数
  - それはそう

### 6.4

- Managerクラスとしてのシングルトンをむやみに作ってはいけない
  - BulletManagerの例
  - 自分でできることは自分でやらせる
    - 自分でできない事(インスタンスが知り得ない外部環境を使って次の行動を決定するなど)は、サービスクラスを使えば良いのではないか？

### シングルトンの代替案

- 引数渡し
  - 古典的だが、有効な事もある
  - メソッドの処理そのものに関連が全くないのならば別の方法を考える必要があるが、そうでないならばこれで良い
- 基底クラスに持たせる
  - 例えば、Player、Enemy、Bulletの基底クラスEntityに、ログ出力を行うためのクラスLogを、protectedで持たせる
    - では次に、どうやってEntityにLogのインスタンスを渡すのか、という話になるが、その解決策として、「参照を渡すための初期化関数を用意する」「サービスロケータ(16章参照)を使う」などがある
- シングルトンをまとめ上げるシングルトンを作る
  - SoudnManager、LogManagerを、それぞれ別々のシングルトンにするのではなく、一個のシングルトンGameのフィールド変数にする

## 7章　Stateパターン

- アクションゲームなどで、複雑な状態遷移をフラグで管理するのは不可能
  - そこでStateパターンを使う

- Stateパターンに入口処理と出口処理を追加する
  - Stateの切り替え時に、前のStateの出口処理と後のStateの入口処理を呼び出す

- 単純なStateパターンでは、並列状態機械を作りにくい
  - 「銃を持つ」「剣を持つ」「ジャンプする」「かがむ」という動作ができるPlayerを考える
  - 実際には、「銃を持ちながらジャンプする」「剣を持ちながらジャンプする」など、Stateは動作の組み合わせになり、その総数は動作が増えるに従って爆発的に増加する
  - 解決策として、2つの状態機構を並列に動作させる
    - 今回の例では、「動作用State機構」と「所持武器用State機構」を用意し、別々のサイクルで回す
    - この方法は、2つの機構が独立ならばうまく動作する
    - しかし実際には、「ジャンプ中は発砲できない」など相互に関連しあう事がほとんど
    - その場合は、if文相方の状態をチェックするなどで対処するしかない

- 階層的状態機械について
  - Stateは、親State、子Stateを定義する事が可能
  - Updateで、自分で処理できない部分は親Stateに処理を任せてしまう
    - 継承したメソッドのオーバーライドに似ている

- プッシュダウンオートマトン
  - 普通のオートマトンだと、遷移前の状況を知らない
  - 一方こちらでは、遷移前の状態に戻ることができる
  - 実装する動作は、「新しいStateをpushして遷移する」「現在のStateをpopして前の状態に遷移する」の2つ

## 8章 ダブルバッファ

- 低レイヤーでのグラフィック処理について
- 書き込み用、読み込み用のバッファを作り、片方が読み込みをやっている間にもう片方が書き込むを行うようにする
