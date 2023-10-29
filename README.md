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

### コンテナ（Panel）

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

### コントロール

* TextBlock：テキスト表示用
* DataGrid：テーブル表示
* ListBox；リスト表示
  * ItemsSource：リスト内容
  * ListBox.ItemContainerStyle：全体のデザイン
  * ListBox.ItemTemplate：各項目のデザイン
    * DataTemplate：項目の内容部viewのテンプレート
* RadioButton:複数のRadioButtonから1つを選択する
  * GroupName：RadioButtonをまとめる（この１グループから1つが選ばれる）
* Calendar：カレンダー

### シェイプ

* Border：横線
* Rectangle：矩形
* Ellipse：楕円

### メディア

* Image　画像
* MediaElement　動画や音声を再生

### ドキュメント

*

## XAMLのお約束

[参照](https://atmarkit.itmedia.co.jp/ait/articles/1006/22/news101.html)

Microsoftが提唱するCLR（Common Language Runtime）オブジェクトをXMLべースで記述する。
CLRオブジェクトは、C#やVBでも記述できる。
```
namespace Sample
{
  public class Point
  {
    public int X { get; set; }
    public int Y { get; set; }
  }
}

var ... = new Sample.Point { X = 1, Y = 2 };
```
が
```
<Point X="1" Y="2"
  xmlns="clr-namespace:Sample;assembly=..." />
```

プロパティの値に{x:Static } 記述を行うとMemberでstatic変数参照ができる。
```
"{x:Static Member=SystemColors.HighlightTextBrush}"
```

XAMLを使うか、コーディングするか、どちらでもよい。

* Childrenプロパティが子要素を管理する（UIElementCollectionクラス）

```
public class MainWindow : Window
{
  public MainWindow()
  {
    this.Content =
      new Grid
      {
        Children = {
          new StackPanel {
            Children = {
              new Menu { Items =
                { new MenuItem { Header = "メニュー" } } },
              new Button { Content = "ボタン" },
              new CheckBox { Content = "チェックボックス" },
              new ComboBox { Items =
                { "コンボボックス" }, SelectedIndex = 0 },
              new RadioButton { Content = "ラジオボタン" },
              new Slider(),
              new ListBox { Items = {
                  "リストボックス項目1",
                  "リストボックス項目2",
                }}}}}};
  }
```
通常

* XXX.xaml
  *  XXX.xaml.cs

のペアで画面を記述し、中間ファイル

* XXX.baml
* XXX.g.cs

が作成され、そこからオブジェクトファイルとなる。

```
<Grid
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  x:Class="Sample.MyGrid">
  <Button
    x:Name="button1"
    Content="ボタン"
    Click="button1_Click" />
</Grid>
```
x:Class属性やx:Name属性もその一種で、x:Class属性には分離コードで実装するクラス名を、x:Name属性にはフィールド名を指定する。
↓
```
namespace Sample
{
  public partial class MyGrid : Grid
  {
    public MyGrid()
    {
      InitializeComponent();
    }

    void button1_Click(object sender, System.Windows.RoutedEventArgs e)
    {
      MessageBox.Show("ボタンが押されました");
    }
  }
}
```
### プロパティ要素構文（property element syntax）
```
<Button Background="Blue" />
```
と文字列で指定できない属性はXML属性の代わりに、子要素としてプロパティ値を設定する
```
<Button>
  <Button.Background>Blue</Button.Background >
</Button>
```

### マークアップ拡張（markup extension）
これがデータバインディングの肝となる。
XML属性中に { } で囲った記述を書くと、対応するマークアップ拡張クラスのインスタンス生成と、そのProvideValueメソッド呼び出しを通してプロパティの値が設定される。
```
  <TextBox Text="{Binding ElementName=slider, Path=Value}" />
```
が
```
var binding = new System.Windows.Data.Binding
{
  ElementName = "slider",
  Path = new PropertyPath("Value"),
};
```
となる。
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
