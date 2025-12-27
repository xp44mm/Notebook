Object
  │
  └── DispatcherObject
        │
        └── DependencyObject
              │
              └── Visual
                    │
                    └── UIElement
                          │
                          └── FrameworkElement
                                │
                                ├── Panel (布局面板基类)
                                │   ├── Canvas
                                │   ├── DockPanel
                                │   ├── Grid
                                │   ├── StackPanel
                                │   ├── UniformGrid
                                │   ├── VirtualizingPanel
                                │   │   ├── VirtualizingStackPanel
                                │   │   └── WrapGrid
                                │   └── WrapPanel
                                │
                                ├── Control (控件基类)
                                │   ├── ContentControl
                                │   │   ├── ButtonBase
                                │   │   │   ├── Button
                                │   │   │   ├── RepeatButton
                                │   │   │   └── ToggleButton
                                │   │   │       ├── CheckBox
                                │   │   │       └── RadioButton
                                │   │   ├── Frame
                                │   │   ├── GroupItem
                                │   │   ├── Label
                                │   │   ├── ListBoxItem
                                │   │   ├── NavigationWindow
                                │   │   ├── StatusBarItem
                                │   │   ├── ToolTip
                                │   │   └── Window
                                │   │
                                │   ├── HeaderedContentControl
                                │   │   ├── Expander
                                │   │   ├── GroupBox
                                │   │   └── TabItem
                                │   │
                                │   ├── ItemsControl
                                │   │   ├── ComboBox
                                │   │   ├── ContextMenu
                                │   │   ├── DataGrid
                                │   │   ├── ItemsPresenter
                                │   │   ├── ListBox
                                │   │   ├── ListView
                                │   │   ├── Menu
                                │   │   ├── MenuBase
                                │   │   │   ├── ContextMenu
                                │   │   │   └── Menu
                                │   │   ├── MultiSelector
                                │   │   │   └── DataGrid
                                │   │   ├── Selector
                                │   │   │   ├── ComboBox
                                │   │   │   ├── ListBox
                                │   │   │   └── TabControl
                                │   │   ├── StatusBar
                                │   │   ├── TabControl
                                │   │   ├── TreeView
                                │   │   └── TreeViewItem
                                │   │
                                │   ├── RangeBase
                                │   │   ├── ProgressBar
                                │   │   ├── ScrollBar
                                │   │   └── Slider
                                │   │
                                │   ├── TextBoxBase
                                │   │   ├── PasswordBox
                                │   │   ├── RichTextBox
                                │   │   └── TextBox
                                │   │
                                │   └── 其他控件
                                │       ├── Calendar
                                │       ├── DatePicker
                                │       ├── DocumentViewer
                                │       ├── InkCanvas
                                │       ├── MediaElement
                                │       ├── Page
                                │       ├── PasswordBox
                                │       ├── ProgressBar
                                │       ├── ScrollViewer
                                │       ├── Separator
                                │       ├── Thumb
                                │       └── WebBrowser
                                │
                                ├── Shape (图形基类)
                                │   ├── Ellipse
                                │   ├── Line
                                │   ├── Path
                                │   ├── Polygon
                                │   ├── Polyline
                                │   └── Rectangle
                                │
                                ├── TextElement
                                │   ├── Block
                                │   │   ├── Paragraph
                                │   │   ├── Section
                                │   │   └── Table
                                │   └── Inline
                                │       ├── AnchoredBlock
                                │       ├── InlineUIContainer
                                │       ├── LineBreak
                                │       ├── Run
                                │       └── Span
                                │
                                └── 其他重要派生类
                                    ├── Adorner
                                    ├── Border
                                    ├── BulletDecorator
                                    ├── DocumentPageView
                                    ├── DocumentReference
                                    ├── Figure
                                    ├── FixedPage
                                    ├── FloatingWindow
                                    ├── Glyphs
                                    ├── Image
                                    ├── InkPresenter
                                    ├── PageContent
                                    ├── Popup
                                    ├── TextBlock
                                    ├── Viewbox
                                    └── Viewport3D