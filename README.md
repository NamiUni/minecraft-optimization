
# Minecraftサーバー最適化ガイド

バージョン1.16.5のガイド

[このガイド](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) と他の情報源に基づいています (これらの情報源は、関連する場合はガイド全体にリンクされています)。

このガイドは、上の方にある目次(`README.md`の次)から簡単に読み進めることができます。

## はじめに
完璧な結果を得るためのガイドは存在しません。それぞれのサーバーには独自のニーズがあり、どれだけの犠牲を払えるか、あるいは払えるかの限界があります。オプションをいじって、サーバーのニーズに合わせて微調整することがすべてです。このガイドは、どのオプションがパフォーマンスに影響を与えるのか、何を変更するのかを理解していただくことを目的としています。このガイドで不正確な情報を見つけた場合は、issueを開いたり、プルリクエストを送ることができます。

## 準備

### Server jar
サーバーソフトウェアの選択によって、パフォーマンスやAPIの可能性に大きな違いが生まれます。現在、人気のあるserver jarは複数ありますが、様々な理由から避けた方がいいものもあります。

推奨ソフトウェア:
* [Paper](https://github.com/PaperMC/Paper) - これはより一般的なサーバーソフトウェアで、パフォーマンスを向上させると同時に、ゲームプレイやメカニックの不整合を修正することを目的としています。
* [Tuinity](https://github.com/Spottedleaf/Tuinity) - 紙よりもさらにサーバー性能の向上を目指した紙のフォーク
* [Airplane](https://github.com/Technove/Airplane) - tuinityのフォークで、tuinityよりもさらにサーバーのパフォーマンスを向上させることを目的としています。
* [Purpur](https://github.com/pl3xgaming/Purpur) - airplaneのフォークで、サーバーのオーナーに機能設定の自由度を与えることを目的としています。

関わらない方がいい:
* Yatopia - "不安定さと保守性を最大限に高めるために、Paper forksの力を結集しました！！" - [KennyTVの恥リスト](https://github.com/KennyTV/list-of-shame) これ以上、何も言うことはありません。
* 非同期を謳っている有料のserver jarは、99.99%の確率で詐欺です。
* Bukkit/Craftbukkit/Spigot - あなたが利用できる他のサーバーソフトウェアと比較して、パフォーマンスの点で非常に時代遅れです。
実行時にプラグインの有効化/無効化/再読み込みを行うあらゆるプラグイン/ソフトウェア。理由は[このセクション](#plugins-enablingdisabling-other-plugins)を参照してください。

### マップの事前生成
マップの事前生成は、低予算のサーバーを改善するための最も重要なステップの1つです。共有CPUやシングルコアのノードでホストされているサーバーでは、非同期のチャンクローディングを十分に利用できないため、この方法が最も有効です。[chunky](https://github.com/pop4959/Chunky)のようなプラグインを使って、ワールドを事前に生成することができます。プレイヤーが新しいチャンクを生成しないように、必ずワールドボーダーを設定してください。事前生成プラグインで設定した半径によっては、事前生成に数時間かかることがあるので注意してください。

ここで重要なのは、オーバーワールド、ネザー、ジ・エンドにはそれぞれ別のワールドボーダーがあり、それぞれのワールドに合わせて設定する必要があるということです。ネザーワールドはオーバーワールドの8倍の大きさなので(データパックで修正されていない場合)、サイズの設定を間違えるとプレイヤーがワールドボーダーの外に出てしまうかもしれません。

**必ずバニラのワールドボーダーを設定してください(`/worldborder set [radius]`)。これは、ラグの原因となる宝の地図の検索範囲など、特定の機能を制限するためです。**

## 設定

### ネットワーク構築

#### [`server.properties`]

##### network-compression-threshold
これにより、サーバが圧縮を試みる前に、パケットサイズの上限を設定することができます。この値を高く設定すると、帯域幅を犠牲にしてCPUリソースを節約することができ、-1に設定するとこの機能は無効になります。この値を高く設定すると、ネットワーク接続が遅いクライアントに悪影響を及ぼす可能性があります。サーバーがプロキシ付きのネットワークにある場合や、同じマシン上にある場合（pingが2ms以下の場合）、内部ネットワークの速度は通常、追加の圧縮されていないトラフィックを処理できるため、この機能（-1）を無効にすることは有益です。

#### [`purpur.yml`]

##### use-alternate-keepalive
Purpurの代替キープアライブシステムを有効にすることで、接続不良のプレイヤーがタイムアウトになることが少なくなります。TCPShieldとの互換性がありません。

> これを有効にすると、プレイヤーに1秒に1回キープアライブパケットを送信し、30秒以内に1つも応答がなかった場合にのみタイムアウトのキックを行います。30秒以内に応答がなかった場合のみ、タイムアウトでキックされます。どのような順番であっても、応答があればプレイヤーの接続は維持されます。つまり、1つのパケットがどこかでドロップしたからといって、プレイヤーがキックされることはありません。 
~ https://pl3xgaming.github.io/PurpurDocs/Configuration/#use-alternate-keepalive

---

### チャンク

#### [`spigot.yml`]

##### view-distance
View-distanceとは、サーバーがtickを消費することになる、プレイヤーの周囲のチャンク単位の距離です。基本的には、プレイヤーから何かが起こるまでの距離です。これには炉の精錬、作物や苗木の成長などが含まれます。この値は [`server.properties`] の値を上書きし、ワールドごとに設定できるので、[`spigot.yml`] で設定してください。これは、`no-tick-view-distance` の存在から、意図的に低く設定したいオプションで、`3`や`4`あたりに設定します。no-tickは、プレイヤーがtickしないでより多くのチャンクをロードすることを可能にします。これにより、プレイヤーはパフォーマンスに影響を与えることなく、より遠くまで見ることができます。

#### [`paper.yml`]

##### no-tick-view-distance
このオプションでは、プレイヤーが描画することのできる最大距離をチャンク単位で設定することができます。これにより、`view-distance`を低く設定しても、プレイヤーにもっと遠くまで描画してもらうことができます。重要なのは、実際の `view-distance` を超えるチャンクは表示されませんが、ストレージからはロードされるので、やりすぎないようにしてください。基本的には `10` が設定すべき最大値です。今のところ、チャンクはビューディスタンスの設定に関わらずクライアントに送信されるので、このオプションの値を高くすると、接続速度の遅いプレイヤーに問題が発生する可能性があります。

##### delay-chunk-unloads-by
このオプションでは、プレイヤーが去った後にチャンクがロードされ続ける時間を設定できます。これにより、プレイヤーが前後に移動しても、同じチャンクを常にロードしたりアンロードしたりしないようになります。値が高すぎると、一度に多くのチャンクがロードされてしまいます。頻繁にテレポートされてロードされるエリアでは、そのエリアを恒久的にロードすることを検討してください。常にチャンクをロードしたりアンロードしたりするよりも、サーバーの負担が軽くなります。

##### max-auto-save-chunks-per-tick
平均的なパフォーマンスを向上させるために、タスクを時間的に分散させることで、ワールドセーブの増分を遅くすることができます。20～30人以上のプレイヤーがいる場合は、`8`よりも高く設定するとよいでしょう。増分保存が間に合わなかった場合、bukkit は自動的に残りのチャンクを一度に保存し、再度プロセスを開始します。

##### prevent-moving-into-unloaded-chunks
この機能を有効にすると、プレイヤーがロードされていないチャンクに移動した際に同期ロードが発生し、メインスレッドに負担をかけてラグが発生するような事を防ぐことができます。no-tick-view-distanceが低いほど、プレイヤーがアンロードされたチャンクにつまずく確率が高くなります。

##### entity-per-chunk-save-limit
このエントリでは、指定したタイプのエンティティを1チャンク内に保存できる数に制限を設けることができます。大量の発射物が保存され、それをロードする際にサーバーがクラッシュするという問題を避けるために、少なくとも発射物ごとに制限を設ける必要があります。以下に、すべての発射物のリストを示します。お好みで制限値を調整してください。すべての発射物の推奨値は `10` 程度です。また、タイプ名で他のエンティティをこのリストに追加することもできます。この設定オプションは、プレイヤーが大規模なモブファームを作るのを防ぐためのものではありません。
```
entity-per-chunk-save-limit:
    arrow: -1
    dragonfireball: -1
    egg: -1
    ender_pearl: -1
    fireball: -1
    firework: -1
    largefireball: -1
    lingeringpotion: -1
    llamaspit: -1
    shulkerbullet: -1
    sizedfireball: -1
    snowball: -1
    spectralarrow: -1
    splashpotion: -1
    thrownexpbottle: -1
    trident: -1
    witherskull: -1
```

##### armor-stands-tick
ほとんどの場合、これを安全に `false` に設定することができます。アーマースタンドやその動作を変更するプラグインを使用していて問題が発生した場合は、再度有効にしてください。これにより、アーマースタンドが水に押されたり、重力の影響を受けたりしなくなります。

##### armor-stands-do-collision-entity-lookups
ここでは、アーマースタンドの衝突を無効にすることができます。これは、アーマースタンドがたくさんあって、何かと衝突する必要がない場合に役立ちます。

---

### モブ

#### [`bukkit.yml`]

##### spawn-limits
MOBを制限する計算は「`[playercount] * [limit]`」で、「playercount」はサーバー上の現在のプレイヤーの数です。論理的には、数字が小さければ小さいほど、MOBの数は少なくなります。`per-player-mob-spawn`は、これに追加の制限を加えて、MOBがプレイヤー間で均等に分配されるようにします。これを減らすことは両刃の剣です。確かにサーバーの負担は減りますが、ゲームモードによっては自然にスポーンするMOBがゲームプレイの大きな部分を占めています。`mob-spawn-range`を適切に調整すれば、20以下にすることもできます。`mob-spawn-range`を低く設定すると、各プレイヤーの周りにもっと多くのMOBがいるように感じられます。tuinityを使用している場合は、[`tuinity.yml`]でワールドごとのMOB制限を設定できます。
##### ticks-per
このオプションは、サーバーが特定のエンティティをスポーンしようとする頻度（tick）を設定します。Water/ambient mobsは、通常そんなに早く殺されることはないので、1tickごとにスポーンする必要はありません。モンスターの場合はMOBファーム(トラップタワー)であっても、スポーン間の時間をわずかに長くしても、スポーン率に影響はありません。ほとんどの場合、このオプションのすべての値は`1`よりも高く設定する必要があります。この値を高くすることで、MOBのスポーンが禁止されている地域でもサーバーがうまく対応できるようになります。

#### [`spigot.yml`]

##### mob-spawn-range
プレイヤーの周りにいるMOBがスポーンする範囲を（チャンク単位で）狭めることができます。サーバーのゲームモードやプレイカウントによっては、[`bukkit.yml`]の`spawn-limits`と一緒にこの値を下げたほうがいいかもしれません。この値を小さくすると、周りにいるモブの数が増えたように感じられます。この値はview distanceと同じかそれ以下で、 hard despawn range/16 より大きくしてはいけません。

##### entity-activation-range
エンティティがtickする（何かをする）ために、プレイヤーからどのくらいの距離が必要かを設定できます。この値を下げるとパフォーマンスが向上しますが、プレイヤーが近づかないと反応しないMOBが出てきます。この値を下げすぎると、特定のMOBファームが壊れる可能性があります（鉄インゴットTTが最も多い犠牲者です）。

##### entity-tracking-range
これは、エンティティが見えるようになるまでのブロックの距離です。プレイヤーには送信されません。この値が低すぎると、プレイヤーの近くに突然モブが現れるようになることがあります。ほとんどの場合、この値は`entity-activation-range`よりも高く設定する必要があります。

##### tick-inactive-villagers
これを有効にすることにより、activation range外で村人をチェックするかどうかをコントロールすることができます。これにより、村人は通常通りに行動し、activation rangeを無視します。これを無効にすると、パフォーマンスが向上しますが、特定の状況下ではプレイヤーが混乱する可能性があります。これにより、鉄インゴットTTや交易の補充に関する問題が発生する可能性があります。

##### nerf-spawner-mobs
スポナーブロックでスポーンされたMOBはAIを持たないようにできます。ナーフされたMOBは何もしません。[`paper.yml`]の`spawner-nerfed-mobs-should-jump`を`true`に変更することで、水中でジャンプするようにすることができます。

#### [`paper.yml`]

##### despawn-ranges
エンティティのデスポーン範囲（ブロック単位）を調整できます。これらの値を低くすると、プレイヤーから遠くにいるMOBをより早くデスポーンさせることができます。soft rangeは`30`程度にして、hard rangeは実際の描画距離より少し多めに調整してください。プレイヤーがチャンクがロードされているポイントを超えたときにMOBがすぐにデスポーンしないようにします(これは[`paper.yml`]の`delay-chunk-unloads-by`でうまく機能しています)。MOBがhard rangeから外れると、即座にデスポーンされます。soft rangeとhard rangeの間にいるときは、ランダムな確率でデスポーンされます。ほとんどの場合、hard rangeはsoft rangeよりも大きくする必要があります。

##### per-player-mob-spawns
このオプションは、ターゲットプレイヤーの周りにいるMOBの数をモブスポーンに考慮するかどうかを決定します。プレイヤーがMOBファームを作ってMOBキャップ全体を占有しているために、MOBのスポーンが一貫していないという問題を回避することができます。これにより、シングルプレイヤーのようなスポーン体験が可能になり、`spawn-limits`を低く設定することができます。この機能を有効にすると、パフォーマンスにごくわずかな影響がありますが、その影響は`spawn-limits`の改善によって相殺されます。

##### max-entity-collisions
[`spigot.yml`]の同名のオプションを上書きします。1つのエンティティが一度に処理できる衝突の回数を決めることができます。`0`を指定すると、プレイヤーを含む他のエンティティを押すことができなくなります。ほとんどの場合、`2`の値で十分でしょう。

##### update-pathfinding-on-block-update
これを無効にすると、経路探索の回数が減り、パフォーマンスが向上します。MOBは5ティック（0.25秒）ごとに経路を受動的に更新します。

##### fix-climbing-bypassing-cramming-rule
これを有効にすると、エンティティーが登っているときに詰め込みの影響を受けないように修正されます。これにより、登っていても狭い場所に無茶苦茶な量のモブが積まれることがなくなります（クモ）。

#### [`airplane.air`]

##### max-tick-freq
このオプションは、プレイヤーから最も遠い場所にあるエンティティがティックされる最も遅い時間を定義します。この値を大きくすると、視界から遠いエンティティのパフォーマンスが向上しますが、MOBファームが壊れたり、MOBの動作が大幅に弱くなる可能性があります。

##### activation-dist-mod
モブをtickする際のグラデーションをコントロールします。DEARはEARのようなハードなカットオフではなく、グラデーションで動作します。近くのエンティティを完全にティックし、遠くのエンティティをほとんどティックしない代わりに、DEARはこの計算結果に基づいてエンティティがティックされる量を減らします。これを減少させると、DEARはプレイヤーの近くで起動し、DEARのパフォーマンスが向上しますが、エンティティが周囲とどのように相互作用するかに影響し、MOBファームが壊れる可能性があります。

#### [`purpur.yml`]

##### dont-send-useless-entity-packets
このオプションを有効にすると、サーバーが空の位置変更パケットを送信するのを防ぐことで、帯域幅を節約できます（デフォルトでは、サーバーは、エンティティが移動していなくても、各エンティティに対してこのパケットを送信します）。クライアント側のエンティティを使用するプラグインで問題が発生する可能性があります。

##### aggressive-towards-villager-when-lagging
これを有効にすると、サーバーが [`purpur.yml`] の `lagging-threshold` で設定された tps のしきい値を下回ると、ゾンビは村人をターゲットにするのをやめます。

##### entities-can-use-portals
このオプションは、プレイヤー以外のすべてのエンティティのポータル使用を無効にすることができます。これにより、メインスレッドで処理されるWorldの変更によってエンティティがチャンクをロードすることを防ぎます。これは、エンティティがポータルを通過できなくなるという副作用があります。

##### villager.brain-ticks
このオプションでは、村人の頭脳（workとpoi）がどのくらいの頻度（tick）で刻むかを設定できます。`3`以上にすると、村人の動きが不安定になったり、バグったりすることが確認されています。

##### villager.lobotomize
ロボトミー化した村人はAIが剥奪され、たまにしか交易アイテムを補充しません。これを有効にすると、目的地までの経路探索ができない村人をロボトミー化します。解放するとロボトミー化が解除されます。

---

### その他

#### [`spigot.yml`]

##### merge-radius
合体するアイテムや経験値オーブの距離を決めて、地面でtickするアイテムの量を減らします。この値を高くしすぎると、アイテムやExpオーブが合体して消えてしまうような錯覚に陥ります。高すぎると一部の農場が破壊されたり、アイテムがブロックをテレポートしたりします。アイテムが壁を越えて合体するのを防ぐためのチェックは行われません。経験値は作成時にのみ合体します。

##### hopper-transfer
ホッパーがアイテムを移動するのを待つ時間（tick）です。ホッパーの数が多い場合はこの値を大きくすることでパフォーマンスが向上しますが、大きくしすぎるとホッパーを使った時計やアイテムのソートシステムが壊れる可能性があります。

##### hopper-check
ホッパーが上のアイテムや上のインベントリのアイテムをチェックするまでの時間をtickで表します。この値を大きくすると、サーバーに多くのホッパーがある場合にはパフォーマンスが向上しますが、ホッパーを使った時計や水流に依存したアイテムソートシステムは壊れます。

#### [`paper.yml`]

##### alt-item-despawn-rate
このリストでは、特定の種類のドロップアイテムのデスポーンにかかる代替時間を、デフォルトよりも速くまたは遅く設定することができます。このオプションは、アイテム消去プラグインの代わりに `merge-radius` と一緒に使うことで、パフォーマンスを向上させることができます。

##### use-faster-eigencraft-redstone
これを有効にすると、レッドストーンシステムがより高速な代替バージョンに置き換えられ、冗長なブロックアップデートが減り、サーバーが行う作業量が減少します。これを有効にすると、ゲームプレイの不整合を引き起こすことなく、パフォーマンスを大幅に向上させることができます。これを有効にすると、craftbukkitからのいくつかのレッドストーンの不整合も修正されます。

##### disable-move-event
`InventoryMoveItemEvent`は、そのイベントをアクティブにリッスンしているプラグインが存在しない限り発生しません。つまり、そのようなプラグインがあり、そのプラグインがこのイベントに対応できなくても気にしない場合にのみ、これを true に設定する必要があります。**保護プラグインなど、このイベントをリッスンするプラグインを使用したい場合は、trueに設定しないでください！**。

##### mob-spawner-tick-rate
このオプションでは、スポナーがtickされる頻度を設定できます。高い値を設定すると、多くのスポナーがある場合にラグが少なくなりますが、（スポナーの遅延と比較して）高すぎる値を設定すると、MOBのスポーン率が低下します。

##### optimize-explosions
これを`true`に設定すると、バニラの爆発アルゴリズムをより高速なものに置き換えますが、その代償として爆発ダメージの計算が若干不正確になります。これは通常、目立たないものです。

##### enable-treasure-maps
宝の地図を生成するのは非常にコストがかかり、探し出そうとする構造物が事前に生成されたワールドの外にある場合、サーバーがハングアップする可能性があります。事前にワールドを生成し、バニラワールドの境界線を設定している場合にのみ、この機能を有効にすることができます。

##### treasure-maps-return-already-discovered
このオプションのデフォルト値は、新しく生成されたマップに未探索の構造物を探させるもので、通常は事前に生成された地形の外にあります。このオプションを true に設定すると、マップは以前に発見された構造物につながるようになります。このオプションを`true`にしないと、新しい宝の地図を生成するときにサーバーがハングアップしたり、クラッシュしたりすることがあります。

##### grass-spread-tick-rate
サーバーが草や菌糸を広げようとするまでの時間（単位：tick）です。これにより、広い範囲の土が草や菌糸に変わるまでに少し時間がかかるようになります。この値を `4` 程度に設定すると、拡散率の低下を目立たせずにうまくいくでしょう。

##### container-update-tick-rate
コンテナの更新間隔をtickで表した時間。これを増やすと、コンテナの更新で問題が発生した場合には助けになるかもしれませんが（滅多にありません）、プレイヤーがインベントリ（ゴーストアイテム）を操作したときにdesyncが発生しやすくなります。

##### non-player-arrow-despawn-rate
MOBが放った矢が何かに当たった後に消えるまでの時間をティックで指定します。プレイヤーは矢を拾うことができないので、`20`(1秒)のように設定するとよいでしょう。

##### creative-arrow-despawn-rate
クリエイティブモードでプレイヤーが放った矢が何かに当たった後に消えるまでの時間をティックで指定します。プレイヤーは矢を拾うことができないので、`20`（1秒）のように設定するとよいでしょう。

#### [`purpur.yml`]

##### disable-treasure-searching
イルカが宝の地図のような構造検索をするのを防ぐ

##### teleport-if-outside-border
プレイヤーがワールドボーダーの外にいた場合、スポーン地点にテレポートできるようにします。バニラのワールドボーダーは迂回可能で、プレイヤーに与えるダメージを軽減することができるので便利です。

---

### ヘルパー

#### [`paper.yml`]

##### anti-xray
これを有効にすると、x-rayの使用者から鉱石を隠すことができます。この機能の詳細な設定については、[Stonar96's recommended settings](https://gist.github.com/stonar96/ba18568bd91e5afd590e8038d14e245e)を参照してください。これを有効にすると、実際にはパフォーマンスが低下しますが、どのx-ray対策プラグインよりもはるかに効率的です。ほとんどの場合、パフォーマンスへの影響は無視できるでしょう。

##### remove-corrupt-tile-entities
タイルエンティティに関するエラーでコンソールがスパムされている場合は、これを `true` に変更してください。これはエラーの原因となっているタイルの実体を無視するのではなく、削除します。タイルエンティティに関する警告が頻繁に出る場合は、なぜ壊れているのかを調査してください。これは根本的な問題を解決するものではありません。

##### nether-ceiling-void-damage-height
このオプションが`0`より大きい場合、設定されたyレベル以上のプレイヤーは、あたかも奈落にいるかのようにダメージを受けます。これにより、プレイヤーはネザーの屋根を使えなくなります。バニラのネザーの高さは128ブロックなので、`127`に設定するとよいでしょう。何らかの方法でネザーの高さを変更した場合は、これを `[あなたが設定した高さ] - 1` に設定してください。

---

## Java起動設定
[次期バージョン1.17のPaperとそのフォークには、Java 11 (LTS)以上が必要です](https://papermc.io/forums/t/java-11-mc-1-17-and-paper/5615)。2021年に向けて、ついにJavaのバージョンを更新しました。2021年に向けて、ついにJavaのバージョンを更新しましょう！（あるいは、少なくともホストに通知して、ホストが移行を処理できるようにしましょう）。Oracle社はライセンスを変更しており、JavaをOracle社から入手する必要性はもはやありません。推奨されるベンダーは[Amazon Corretto](https://aws.amazon.com/corretto/)と[AdoptOpenJDK](https://adoptopenjdk.net)です。OpenJ9のような代替JVMの実装も動作しますが、これらは紙媒体ではサポートされておらず、問題が発生することが知られているため、現在は推奨されていません。

ガベージコレクタは、大きなガベージコレクタのタスクによって引き起こされるラグのスパイクを減らすように設定することができます。minecraft サーバー用に最適化された起動設定は [こちら](https://mcflags.emc.gs/) [`SOG`] で見つけることができます。この推奨事項は他の jvm の実装では機能しないことに注意してください。

## 「素晴らしすぎる」プラグインたち

### 地上のアイテムを削除するプラグイン
これらは[merge radius](#merge-radius)や[alt-item-despawn-rate](#alt-item-despawn-rateenabled)で置き換えることができるため、絶対に必要ありません。また、正直なところ、これらは基本的なサーバー設定よりも設定が少ないです。また、アイテムを削除しないよりも、アイテムをスキャンして削除する方が、より多くのリソースを消費する傾向があります。

### MOB削除プラグイン
使うことを正当化するのはとても難しいです。自然に発生したエンティティを削除すると、サーバーが常により多くのMOBを発生させようとするため、削除しない場合よりもラグが発生します。唯一の「許容できる」使用例は、大量のスポナーがあるサーバーのスポナー用です。

### プラグインが他のプラグインを 有効/無効 にする
実行時にプラグインを有効にしたり無効にしたりするものは非常に危険です。このようなプラグインをロードすると、トラッキングデータに関わる致命的なエラーが発生する可能性があり、プラグインを無効にすると、依存関係の解消によるエラーが発生する可能性があります。`reload`コマンドにも全く同じ問題があり、[me4502's blog post](https://matthewmiller.dev/blog/problem-with-reload/) で詳しく説明されています。

## 何がラグいのか？- パフォーマンスの測定

### mspt
Paperには、`/mspt`コマンドがあり、サーバーが最近のtickを計算するのにかかった時間を知ることができます。最初の値と2番目の値が50より小さければ、おめでとうございます。あなたのサーバーは遅れていません。もし3番目の値が50を超えていたら、少なくとも1回は時間のかかったティックがあったことを意味します。これは全く普通のことで、時々起こることですので、慌てないでください。

### timings
サーバーが遅延しているときに何が起こっているかを確認するには timings が有効です。timings とは、どの作業に最も時間がかかっているかを正確に把握するためのツールです。これは最も基本的なトラブルシューティングツールであり、ラグに関してヘルプを求めた場合、ほとんどの場合、 timings を尋ねられるでしょう。

サーバーの timings を取得するには、`/timings paste`コマンドを実行して、表示されたリンクをクリックするだけです。このリンクを他の人と共有して、助けてもらうこともできます。また、自分が何をしているのか分からないと、誤読してしまうこともあります。読み方については、詳しい[Aikarによるビデオチュートリアル](https://www.youtube.com/watch?v=T4J0A9l7bfQ)がありますので、そちらをご覧ください。

### spark
[Spark](https://github.com/lucko/spark)は、サーバのCPUやメモリの使用状況をプロファイリングすることができるプラグインです。使い方は[wiki](https://github.com/lucko/spark/wiki/Commands)を参照してください。また、ラグ・スパイクの原因を見つける方法については、[こちら](https://github.com/lucko/spark/wiki/Finding-the-cause-of-lag-spikes)を参照してください。


[`SOG`]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/
[`server.properties`]: https://minecraft.gamepedia.com/Server.properties
[`bukkit.yml`]: https://bukkit.gamepedia.com/Bukkit.yml
[`spigot.yml`]: https://www.spigotmc.org/wiki/spigot-configuration/
[`paper.yml`]:  https://paper.readthedocs.io/en/latest/server/configuration.html
[`purpur.yml`]: https://purpur.pl3x.net/docs
[`tuinity.yml`]: https://github.com/Spottedleaf/Tuinity/wiki/Config
[`airplane.air`]: https://airplane.gg/config
