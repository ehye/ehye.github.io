---
title: UWP开发笔记 获取定位信息
date: 2017-01-26 15:43:02
categories: Tips
tags:
    - UWP
---
首先创建一个叫LocationManager的类
```csharp
public class LocationManager
    {
        public async static Task<Geoposition> GetPostion()
        {
            var accessStatus = await Geolocator.RequestAccessAsync();

            if (accessStatus != GeolocationAccessStatus.Allowed)
                throw new Exception();

            var geolocator = new Geolocator { DesiredAccuracyInMeters = 0 };

            var position = await geolocator.GetGeopositionAsync();

            return position;
        }
    }
```

有了position之后，就可以获取经纬度
*in MainPage.xaml.cs*
```csharp
private async void Button_Click(object sender, RoutedEventArgs e)
{
	var position = await LocationManager.GetPostion();
	
	RootObject myWeather = await OpenWeatherMapProxy.GetWeather(position.Coordinate.Latitude, position.Coordinate.Longitude);
	string icon = String.Format("ms-appx:///Assets/Weather/{0}.png", myWeather.results[0].now.code);
	ResultImage.Source = new BitmapImage(new Uri(icon, UriKind.Absolute));
	ResultTextBlock.Text = myWeather.results[0].location.name + " - "
				+ myWeather.results[0].now.temperature + " - " 
				+ myWeather.results[0].now.text;
}
```
*Geocoordinate.Latitude.get提示已过时，实测发现不影响*

> 更多关于如何使用Device的Geolocater模块，详见：
Geolocator Class
https://msdn.microsoft.com/zh-cn/library/windows/apps/windows.devices.geolocation.geolocator.aspx

> 更详细的使用UWP中maps 和 Location（position）的方法：
获取用户位置
https://msdn.microsoft.com/zh-cn/windows/uwp/maps-and-location/get-location