---
title: UWP开发笔记 汉堡菜单
date: 2016-09-04 13:12:33
categories: Tips
tags:
	- UWP
---
先画出汉堡菜单按钮或返回按钮
```xml
<RelativePanel>
	<Button Name="HamburgerButton"
		FontFamily="Segoe MDL2 Assets"
		Content="&#xE700;"
		Click="HamburgerButton_Click"
		FontSize="36"
		Background="LightGray"
		RelativePanel.AlignLeftWithPanel="True"/>
	<Button Name="Back"
		Visibility="Collapsed"
		FontFamily="Segoe MDL2 Assets"
		Content="&#xE7EA;"
		Click="Back_Click"
		FontSize="36"
		Background="White"
		RelativePanel.RightOf="HamburgerButton"/>
</RelativePanel>
```

然后是菜单列表
```xml
<SplitView Grid.Row="1"
	   Name="HamburgerMeun" 
	   DisplayMode="CompactOverlay"
	   OpenPaneLength="200"
	   CompactPaneLength="56" 
	   IsPaneOpen="False" >
<SplitView.Pane>
	<ListBox SelectionMode="Single"
		SelectionChanged="ListBox_SelectionChanged">
		<ListBoxItem Name="Financial">
			<StackPanel Orientation="Horizontal">
				<TextBlock Text="&#xE10F;"
				   FontFamily="Segoe MDL2 Assets"
				   FontSize="36"></TextBlock>
				<TextBlock Text="Home" 
				   FontSize="24"
				   Margin="20,0,0,0"></TextBlock>
			</StackPanel>
		</ListBoxItem>

		<ListBoxItem Name="Food">
			<StackPanel Orientation="Horizontal">
				<TextBlock Text="&#xE1CE;"
					FontFamily="Segoe MDL2 Assets"
					FontSize="36"></TextBlock>
				<TextBlock Text="Food"
					FontSize="24"
					Margin="20,0,0,0"></TextBlock>
			</StackPanel>
		</ListBoxItem>
	</ListBox>
</SplitView.Pane>
```
接下来是点击事件
```cs
	private void HamburgerButton_Click(object sender, RoutedEventArgs e)
	{
		HamburgerMeun.IsPaneOpen = !HamburgerMeun.IsPaneOpen;
	}
```
*Bob Tabord 的汉堡菜单弹出写法*

第一次看到这样写按钮，很巧妙