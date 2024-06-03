ОПИСАНИЕ ПРОЕКТА

Мы создали DLL-библиотеку на C#, которая предоставляет функции для работы с геоданными в формате GeoJSON

КОД ПРОГРАММЫ

using System;
using System.IO;
using GeoJSON.Net.Geometry;
using GeoJSONL;
namespace GeoData
{
    class Program
    {
        static void Main(string[] args)
        {
            string filePath = "C:\\Users\\Admin\\source\\repos\\FGT\\FGT\\jsonn.txt";  
            var data = GeoJSONLibrary.LoadGeoJSON(filePath);
            var type = GeoJSONLibrary.GetGeometryType(data);
            Console.WriteLine($"Тип геометрии: {type}");
            var coordinates = GeoJSONLibrary.GetCoordinates(data);
            Console.WriteLine("Координаты:");
            foreach (var coord in coordinates)
            {
                Console.WriteLine($"X: {coord.Latitude}, Y: {coord.Longitude}");
            }
            var area = GeoJSONLibrary.CalculateArea(data);
            Console.WriteLine($"Площадь: {area} квадратных метров");
            var point = new Position(10.1, 125.6);
            var nearestFeature = GeoJSONLibrary.FindNearestFeature(data, point);
            Console.WriteLine($"Ближайший объект: {nearestFeature?.Properties["name"]}");
        }
    }
}


РАЗРАБОТЧИКИ

Этот проект был разработан студентами группы ИСП-8: Поздняковым Сергеем и Карабековым Дияром. 


