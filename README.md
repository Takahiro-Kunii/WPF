# WPF
WPFとPrism、Behaviorsに関して

* WPF: Windows Presentation Foundation Microsoftが提供するWindowsアプリ用フレームワーク
* Prism：WPFを実装するために便利なフレームワーク
* Behaviors：クリック以外の各種イベントに対して処理を実装するフレームワーク

## WPFのお約束

* デザイナ専用のデータモデル指定ができる
```
  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
  mc:Ignorable="d"

  d:DataContext="{d:DesignInstance local:MainViewModel, IsDesignTimeCreatable=True}">
```

## 組み込みViewについて

### レイアウト用（Panel）

[参照](https://learn.microsoft.com/ja-jp/dotnet/desktop/wpf/controls/panels-overview?view=netframeworkdesktop-4.8)

* Grid：子viewを格子配置
* StackPanel：子viewを水平方向または垂直方向の単一行に配置
* DockPanel：子viewを上下左右に配置
  * DockPanel.Dockで上下左右を指定積み重ねること可能
  ```
            <... DockPanel.Dock="Bottom">
  ```
* ContentControl：任意の種類のコンテンツが 1 つあるコントロール
* Canvas  Canvas 領域に対する座標により子要素を明示的に配置できる領域が定義されています。
* UniformGrid	格子内の子viewをすべて同じセル サイズで整列する
* WrapPanel	左から右へ順に子viewが配置され、外側のボックスの端で改行して次の行へ。後続の配置は、Orientation プロパティの値に応じて、上から下または右から左に向かう。
* TabPanel	TabControl でのタブ ボタンのレイアウトが処理されます。
* ToolBarOverflowPanel	ToolBar コントロール内のコンテンツが整列されます。
* VirtualizingPanel	大量の子viewを表示している部分だけ実体化することで、利用効率、表示効率を上げる。ListViewの派生元？
* VirtualizingStackPanel	水平方向または垂直方向の単一行でコンテンツを整列し、仮想化。DataGridの派生元？

### 基本部品

* TextBlock：テキスト表示用
* ListBox；リスト表示
  * ItemsSource：リスト内容
  * ListBox.ItemContainerStyle：全体のデザイン
  * ListBox.ItemTemplate：各項目のデザイン
    * DataTemplate：項目の内容部viewのテンプレート
* Border：横線
* RadioButton:複数のRadioButtonから1つを選択する
  * GroupName：RadioButtonをまとめる（この１グループから1つが選ばれる）


  
## Prismのお約束
* Projectのフォルダ階層をView,ViewModelの識別に利用する
  * prism:ViewModelLocator.AutoWireViewModel="True"で自動化 
```
<Window.DataContext>
    <vm:MainPageViewModel/>
</Window.DataContext>
```
の省略

* 画面切り替え
  * IRegionManagerで切り替える
  ```
    regionManager.RequestNavigate(REGION_NAME, nameof(XView));
  ```
  * IRegionManagerはコンストラクタで受け取る
  ```
          public MainWindowViewModel(IRegionManager regionManager)
            : this()
        {
            this.regionManager = regionManager;

  ```
  * IModuleCatalogへの登録が必要
  ```
        protected override void RegisterTypes(IContainerRegistry containerRegistry)
        {
            containerRegistry.RegisterForNavigation<XView>();
  ```
  
## Behaviorsのお約束

> [参照](https://zenn.dev/takuty/articles/e9d8f065452266)

* ロード後に Command を実行する
```
<i:Interaction.Triggers>
    <i:EventTrigger EventName="Loaded">
        <i:InvokeCommandAction Command="{Binding LoadedCommand}" />
    </i:EventTrigger>
</i:Interaction.Triggers>
```

* ComboBox の選択を変更すると選択した値が表示される
```
<ComboBox
    HorizontalAlignment="Left"
    VerticalAlignment="Top"
    VerticalContentAlignment="Center"
    Height="25"
    Width="170"
    Margin="150,400,0,0"
    SelectedItem="{Binding SelectedItem}">
    <ComboBoxItem Content="ぶどう" />
    <ComboBoxItem Content="りんご" />
    <ComboBoxItem Content="みかん" />
    <ComboBoxItem Content="もも" />
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="SelectionChanged">
            <i:InvokeCommandAction Command="{Binding SelectionChangedCommand}" />
        </i:EventTrigger>
    </i:Interaction.Triggers>
</ComboBox>
```
