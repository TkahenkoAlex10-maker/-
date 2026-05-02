# -
Book Tracker - это GUI-приложение для управления списком прочитанных книг. Программа позволяет: - Добавлять книги с указанием названия, автора, жанра и количества страниц - Фильтровать книги по жанру и количеству страниц - Удалять книги из списка - Сохранять и загружать данные в формате JSON
import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
from datetime import datetime

class BookTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Book Tracker - Трекер прочитанных книг")
        self.root.geometry("900x600")
        
        # Данные книг
        self.books = []
        self.filtered_books = []
        
        # Загрузка данных из JSON
        self.load_data()
        
        # Создание интерфейса
        self.create_input_frame()
        self.create_filter_frame()
        self.create_table_frame()
        
        # Обновление таблицы
        self.refresh_table()
    
    def create_input_frame(self):
        """Фрейм для ввода данных"""
        input_frame = tk.LabelFrame(self.root, text="Добавление новой книги", padx=10, pady=10)
        input_frame.pack(fill="x", padx=10, pady=5)
        
        # Поля ввода
        tk.Label(input_frame, text="Название книги:").grid(row=0, column=0, sticky="w", padx=5, pady=5)
        self.title_entry = tk.Entry(input_frame, width=30)
        self.title_entry.grid(row=0, column=1, padx=5, pady=5)
        
        tk.Label(input_frame, text="Автор:").grid(row=0, column=2, sticky="w", padx=5, pady=5)
        self.author_entry = tk.Entry(input_frame, width=25)
        self.author_entry.grid(row=0, column=3, padx=5, pady=5)
        
        tk.Label(input_frame, text="Жанр:").grid(row=1, column=0, sticky="w", padx=5, pady=5)
        self.genre_entry = tk.Entry(input_frame, width=30)
        self.genre_entry.grid(row=1, column=1, padx=5, pady=5)
        
        tk.Label(input_frame, text="Количество страниц:").grid(row=1, column=2, sticky="w", padx=5, pady=5)
        self.pages_entry = tk.Entry(input_frame, width=25)
        self.pages_entry.grid(row=1, column=3, padx=5, pady=5)
        
        # Кнопка добавления
        self.add_button = tk.Button(input_frame, text="Добавить книгу", command=self.add_book, 
                                    bg="#4CAF50", fg="white", font=("Arial", 10, "bold"))
        self.add_button.grid(row=2, column=0, columnspan=4, pady=10)
    
    def create_filter_frame(self):
        """Фрейм для фильтрации"""
        filter_frame = tk.LabelFrame(self.root, text="Фильтрация", padx=10, pady=10)
        filter_frame.pack(fill="x", padx=10, pady=5)
        
        # Фильтр по жанру
        tk.Label(filter_frame, text="Фильтр по жанру:").grid(row=0, column=0, padx=5, pady=5)
        self.genre_filter = tk.Entry(filter_frame, width=20)
        self.genre_filter.grid(row=0, column=1, padx=5, pady=5)
        
        # Фильтр по страницам
        tk.Label(filter_frame, text="Страниц больше:").grid(row=0, column=2, padx=5, pady=5)
        self.pages_filter = tk.Entry(filter_frame, width=10)
        self.pages_filter.grid(row=0, column=3, padx=5, pady=5)
        
        # Кнопка фильтрации
        self.filter_button = tk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter,
                                       bg="#2196F3", fg="white")
        self.filter_button.grid(row=0, column=4, padx=10, pady=5)
        
        # Кнопка сброса фильтра
        self.reset_button = tk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter,
                                      bg="#FF9800", fg="white")
        self.reset_button.grid(row=0, column=5, padx=5, pady=5)
    
    def create_table_frame(self):
        """Фрейм для таблицы с книгами"""
        table_frame = tk.Frame(self.root)
        table_frame.pack(fill="both", expand=True, padx=10, pady=5)
        
        # Создание Treeview
        columns = ("ID", "Название", "Автор", "Жанр", "Страницы")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=20)
        
        # Настройка колонок
        self.tree.heading("ID", text="ID")
        self.tree.heading("Название", text="Название книги")
        self.tree.heading("Автор", text="Автор")
        self.tree.heading("Жанр", text="Жанр")
        self.tree.heading("Страницы", text="Страницы")
        
        self.tree.column("ID", width=50)
        self.tree.column("Название", width=250)
        self.tree.column("Автор", width=200)
        self.tree.column("Жанр", width=150)
        self.tree.column("Страницы", width=100)
        
        # Скроллбар
        scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
        
        # Размещение
        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")
        
        # Контекстное меню для удаления
        self.context_menu = tk.Menu(self.root, tearoff=0)
        self.context_menu.add_command(label="Удалить книгу", command=self.delete_book)
        self.tree.bind("<Button-3>", self.show_context_menu)
    
    def add_book(self):
        """Добавление новой книги"""
        title = self.title_entry.get().strip()
        author = self.author_entry.get().strip()
        genre = self.genre_entry.get().strip()
        pages = self.pages_entry.get().strip()
        
        # Проверка на пустые поля
        if not title or not author or not genre or not pages:
            messagebox.showerror("Ошибка", "Все поля должны быть заполнены!")
            return
        
        # Проверка корректности количества страниц
        try:
            pages = int(pages)
            if pages <= 0:
                raise ValueError
        except ValueError:
            messagebox.showerror("Ошибка", "Количество страниц должно быть положительным числом!")
            return
        
        # Добавление книги
        book = {
            "id": len(self.books) + 1,
            "title": title,
            "author": author,
            "genre": genre,
            "pages": pages,
            "date_added": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        }
        
        self.books.append(book)
        self.save_data()
        
        # Очистка полей
        self.title_entry.delete(0, tk.END)
        self.author_entry.delete(0, tk.END)
        self.genre_entry.delete(0, tk.END)
        self.pages_entry.delete(0, tk.END)
        
        # Обновление таблицы
        self.refresh_table()
        messagebox.showinfo("Успех", f"Книга '{title}' успешно добавлена!")
    
    def apply_filter(self):
        """Применение фильтрации"""
        genre_filter = self.genre_filter.get().strip().lower()
        pages_filter = self.pages_filter.get().strip()
        
        self.filtered_books = self.books.copy()
        
        # Фильтр по жанру
        if genre_filter:
            self.filtered_books = [book for book in self.filtered_books 
                                   if genre_filter in book["genre"].lower()]
        
        # Фильтр по страницам
        if pages_filter:
            try:
                min_pages = int(pages_filter)
                self.filtered_books = [book for book in self.filtered_books 
                                       if book["pages"] > min_pages]
            except ValueError:
                messagebox.showerror("Ошибка", "Фильтр по страницам должен быть числом!")
                return
        
        self.refresh_table(use_filter=True)
    
    def reset_filter(self):
        """Сброс фильтрации"""
        self.genre_filter.delete(0, tk.END)
        self.pages_filter.delete(0, tk.END)
        self.filtered_books = []
        self.refresh_table()
    
    def refresh_table(self, use_filter=False):
        """Обновление таблицы"""
        # Очистка таблицы
        for item in self.tree.get_children():
            self.tree.delete(item)
        
        # Выбор данных для отображения
        books_to_show = self.filtered_books if use_filter and self.filtered_books else self.books
        
        # Добавление данных в таблицу
        for book in books_to_show:
            self.tree.insert("", "end", values=(book["id"], book["title"], 
                                               book["author"], book["genre"], book["pages"]))
    
    def delete_book(self):
        """Удаление выбранной книги"""
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Предупреждение", "Выберите книгу для удаления!")
            return
        
        # Получение ID книги
        values = self.tree.item(selected[0])["values"]
        book_id = values[0]
        
        # Подтверждение удаления
        if messagebox.askyesno("Подтверждение", f"Удалить книгу '{values[1]}'?"):
            # Удаление книги
            self.books = [book for book in self.books if book["id"] != book_id]
            
            # Перенумерация ID
            for i, book in enumerate(self.books, 1):
                book["id"] = i
            
            self.save_data()
            self.reset_filter()
            messagebox.showinfo("Успех", "Книга удалена!")
    
    def show_context_menu(self, event):
        """Показ контекстного меню"""
        item = self.tree.identify_row(event.y)
        if item:
            self.tree.selection_set(item)
            self.context_menu.post(event.x_root, event.y_root)
    
    def save_data(self):
        """Сохранение данных в JSON"""
        try:
            with open("books.json", "w", encoding="utf-8") as file:
                json.dump(self.books, file, ensure_ascii=False, indent=4)
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить данные: {e}")
    
    def load_data(self):
        """Загрузка данных из JSON"""
        if os.path.exists("books.json"):
            try:
                with open("books.json", "r", encoding="utf-8") as file:
                    self.books = json.load(file)
            except Exception as e:
                messagebox.showerror("Ошибка", f"Не удалось загрузить данные: {e}")
                self.books = []

if __name__ == "__main__":
    root = tk.Tk()
    app = BookTracker(root)
    root.mainloop()
