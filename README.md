ОПИСАНИЕ ПРОЕКТА

Мы создали DLL-библиотеку на C#, которая предоставляет функции для работы с геоданными в формате GeoJSON

КОД ПРОГРАММЫ

```C#
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




using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Newtonsoft.Json;
using GeoJSON.Net.Feature;
using GeoJSON.Net.Geometry;
namespace GeoJSONL
{
    public static class GeoJSONLibrary
    {
        public static GeoJSONData LoadGeoJSON(string filePath)
        {
            string json = File.ReadAllText(filePath);
            var featureCollection = JsonConvert.DeserializeObject<FeatureCollection>(json);
            return new GeoJSONData { FeatureCollection = featureCollection };
        }
        public static void SaveGeoJSON(string filePath, GeoJSONData data)
        {
            string json = JsonConvert.SerializeObject(data.FeatureCollection, Formatting.Indented);
            File.WriteAllText(filePath, json);
        }
        public static string GetGeometryType(GeoJSONData data)
        {
            if (data.FeatureCollection.Features.Count > 0)
            {
                return data.FeatureCollection.Features[0].Geometry.Type.ToString();
            }
            return string.Empty;
        }
        public static List<IPosition> GetCoordinates(GeoJSONData data)
        {
            if (data.FeatureCollection.Features.Count > 0)
            {
                var geometry = data.FeatureCollection.Features[0].Geometry;
                return ExtractCoordinates(geometry);
            }
            return new List<IPosition>();
        }
        public static double CalculateArea(GeoJSONData data)
        {
            if (data.FeatureCollection.Features.Count > 0 && data.FeatureCollection.Features[0].Geometry is Polygon polygon)
            {
                return CalculatePolygonArea(polygon);
            }
            return 0.0;
        }
        public static double CalculateDistance(GeoJSONData data1, GeoJSONData data2)
        {
            if (data1.FeatureCollection.Features.Count > 0 && data2.FeatureCollection.Features.Count > 0)
            {
                var geometry1 = data1.FeatureCollection.Features[0].Geometry as Point;
                var geometry2 = data2.FeatureCollection.Features[0].Geometry as Point;
                if (geometry1 != null && geometry2 != null)
                {
                    return CalculatePointDistance(geometry1.Coordinates, geometry2.Coordinates);
                }
            }
            return 0.0;
        }
        public static Feature FindNearestFeature(GeoJSONData data, IPosition point)
        {
            Feature nearestFeature = null;
            double minDistance = double.MaxValue;
            foreach (var feature in data.FeatureCollection.Features)
            {
                if (feature.Geometry is Point featurePoint)
                {
                    double distance = CalculatePointDistance(point, featurePoint.Coordinates);
                    if (distance < minDistance)
                    {
                        minDistance = distance;
                        nearestFeature = feature;
                    }
                }
            }
            return nearestFeature;
        }
        private static List<IPosition> ExtractCoordinates(IGeometryObject geometry)
        {
            if (geometry is Point point)
            {
                return new List<IPosition> { point.Coordinates };
            }
            else if (geometry is LineString lineString)
            {
                return lineString.Coordinates.Cast<IPosition>().ToList();
            }
            else if (geometry is Polygon polygon)
            {
                return polygon.Coordinates[0].Coordinates.Cast<IPosition>().ToList();
            }
            return new List<IPosition>();
        }
        private static double CalculatePolygonArea(Polygon polygon)
        {
            var coordinates = polygon.Coordinates[0].Coordinates.Cast<Position>().ToList();
            double area = 0.0;

            for (int i = 0, j = coordinates.Count - 1; i < coordinates.Count; j = i++)
            {
                area += (coordinates[j].Longitude + coordinates[i].Longitude) * (coordinates[j].Latitude - coordinates[i].Latitude);
            }
            return Math.Abs(area * 0.5);
        }
        private static double CalculatePointDistance(IPosition pos1, IPosition pos2)
        {
            const double R = 6371e3;
            double φ1 = DegreesToRadians(pos1.Latitude);
            double φ2 = DegreesToRadians(pos2.Latitude);
            double Δφ = DegreesToRadians(pos2.Latitude - pos1.Latitude);
            double Δλ = DegreesToRadians(pos2.Longitude - pos1.Longitude);
            double a = Math.Sin(Δφ / 2) * Math.Sin(Δφ / 2) +
                       Math.Cos(φ1) * Math.Cos(φ2) *
                       Math.Sin(Δλ / 2) * Math.Sin(Δλ / 2);
            double c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
            return R * c;
        }
        private static double DegreesToRadians(double degrees)
        {
            return degrees * Math.PI / 180;
        }
    }
    public class GeoJSONData
    {
        public FeatureCollection FeatureCollection { get; set; }

        public GeoJSONData()
        {
            FeatureCollection = new FeatureCollection();
        }
    }
}


РАЗРАБОТЧИКИ

Этот проект был разработан студентами группы ИСП-8: Поздняковым Сергеем и Карабековым Дияром. 


