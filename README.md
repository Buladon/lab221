using System;
using System.Collections.Generic;
using System.Linq;
using System.Xml.Linq;

public class SubjectIndex
{
    public string Word { get; set; }
    public List<int> PageNumbers { get; set; }

    public SubjectIndex(string word, List<int> pageNumbers)
    {
        Word = word;
        PageNumbers = pageNumbers;
    }
}

public class SubjectIndexManager
{
    private List<SubjectIndex> _indices;

    public SubjectIndexManager()
    {
        _indices = new List<SubjectIndex>();
    }

    // Формирование указателя из файла
    public void LoadFromFile(string filePath)
    {
        var lines = System.IO.File.ReadAllLines(filePath);
        foreach (var line in lines)
        {
            var parts = line.Split(':');
            if (parts.Length == 2)
            {
                var word = parts[0].Trim();
                var pageNumbers = parts[1].Split(',')
                    .Select(p => int.Parse(p.Trim()))
                    .ToList();
                _indices.Add(new SubjectIndex(word, pageNumbers));
            }
        }
    }

    // Печать предметного указателя
    public void PrintIndex()
    {
        foreach (var index in _indices)
        {
            Console.WriteLine($"{index.Word}: {string.Join(", ", index.PageNumbers)}");
        }
    }

    // Сохранение указателя в XML-файл
    public void SaveToXml(string filePath)
    {
        var doc = new XDocument(
            new XElement("SubjectIndex",
                _indices.Select(index =>
                    new XElement("Entry",
                        new XAttribute("Word", index.Word),
                        new XElement("Pages",
                            index.PageNumbers.Select(p => new XElement("Page", p)))))));

        doc.Save(filePath);
    }

    // Вывод номеров страниц для заданного слова
    public List<int> GetPagesForWord(string word)
    {
        var index = _indices.FirstOrDefault(i => i.Word.Equals(word, StringComparison.OrdinalIgnoreCase));
        return index?.PageNumbers ?? new List<int>();
    }

    // Добавление элемента
    public void AddEntry(string word, List<int> pageNumbers)
    {
        if (_indices.Any(i => i.Word.Equals(word, StringComparison.OrdinalIgnoreCase)))
        {
            Console.WriteLine($"Word '{word}' already exists in the index.");
            return;
        }

        _indices.Add(new SubjectIndex(word, pageNumbers));
    }

    // Удаление элемента
    public void RemoveEntry(string word)
    {
        _indices.RemoveAll(i => i.Word.Equals(word, StringComparison.OrdinalIgnoreCase));
    }

    // Изменение объекта
    public void UpdateEntry(string word, List<int> newPageNumbers)
    {
        var index = _indices.FirstOrDefault(i => i.Word.Equals(word, StringComparison.OrdinalIgnoreCase));
        if (index != null)
        {
            index.PageNumbers = newPageNumbers;
        }
    }

    // Сортировка предметного указателя
    public void SortIndex()
    {
        _indices = _indices.OrderBy(i => i.Word).ToList();
    }

    // Фильтрация предметного указателя
    public List<SubjectIndex> FilterIndex(Func<SubjectIndex, bool> predicate)
    {
        return _indices.Where(predicate).ToList();
    }
}

class Program
{
    static void Main()
    {
        var manager = new SubjectIndexManager();

        // Пример использования
        manager.LoadFromFile("input.txt");
        manager.PrintIndex();
        manager.AddEntry("Example", new List<int> { 12, 15 });
        manager.RemoveEntry("OldWord");
        manager.UpdateEntry("Example", new List<int> { 1, 3, 5 });
        manager.SortIndex();

        Console.WriteLine("\nUpdated Index:");
        manager.PrintIndex();

        var pages = manager.GetPagesForWord("Example");
        Console.WriteLine($"\nPages for 'Example': {string.Join(", ", pages)}");

        manager.SaveToXml("output.xml");
    }
}
