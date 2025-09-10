import tkinter as tk
from tkinter import messagebox

notes = []

def clear_window():
    """Удаляет все виджеты из главного окна."""
    for widget in root.winfo_children():
        widget.destroy()

def show_start_screen():
    """Показывает начальный экран с двумя кнопками."""
    clear_window()
    title = tk.Label(root, text="Простое приложение заметок", font=("Arial", 18, "bold"))
    title.pack(pady=20)

    btn_create = tk.Button(root, text="Создать заметку", width=25, height=2, command=show_create_note)
    btn_create.pack(pady=10)

    btn_view = tk.Button(root, text="Просмотреть заметки", width=25, height=2, command=show_view_notes)
    btn_view.pack(pady=10)

    btn_exit = tk.Button(root, text="Выход", width=25, height=2, command=confirm_exit)
    btn_exit.pack(pady=10)

def show_create_note():
    """Экран для создания новой заметки."""
    clear_window()
    label = tk.Label(root, text="Введите текст заметки:", font=("Arial", 14))
    label.pack(pady=10)

    text_entry = tk.Text(root, width=45, height=10)
    text_entry.pack(pady=10)

    def save_note():
        note = text_entry.get("1.0", tk.END).strip()
        if note:
            notes.append(note)
            messagebox.showinfo("Успех", "Заметка сохранена!")
            show_start_screen()
        else:
            messagebox.showwarning("Ошибка", "Заметка не может быть пустой.")

    btn_save = tk.Button(root, text="Сохранить", width=15, command=save_note)
    btn_save.pack(pady=5)

    btn_back = tk.Button(root, text="Назад", width=15, command=show_start_screen)
    btn_back.pack(pady=5)

def show_view_notes():
    """Экран для просмотра и удаления заметок."""
    clear_window()
    label = tk.Label(root, text="Ваши заметки:", font=("Arial", 14))
    label.pack(pady=10)

    if not notes:
        empty_label = tk.Label(root, text="Заметок пока нет.", font=("Arial", 12), fg="gray")
        empty_label.pack(pady=20)
    else:
        frame = tk.Frame(root)
        frame.pack(pady=5, fill="both", expand=True)

        scrollbar = tk.Scrollbar(frame)
        scrollbar.pack(side="right", fill="y")

        listbox = tk.Listbox(frame, width=50, height=10, yscrollcommand=scrollbar.set)
        for i, note in enumerate(notes, 1):
            short_note = note if len(note) < 50 else note[:47] + "..."
            listbox.insert(tk.END, f"{i}. {short_note}")
        listbox.pack(side="left", fill="both", expand=True)
        scrollbar.config(command=listbox.yview)

        def delete_selected():
            selected = listbox.curselection()
            if not selected:
                messagebox.showwarning("Ошибка", "Выберите заметку для удаления.")
                return
            index = selected[0]
            confirm = messagebox.askyesno("Подтверждение", "Удалить выбранную заметку?")
            if confirm:
                del notes[index]
                show_view_notes()

        btn_delete = tk.Button(root, text="Удалить выбранную заметку", width=25, command=delete_selected)
        btn_delete.pack(pady=10)

    btn_back = tk.Button(root, text="Назад", width=25, command=show_start_screen)
    btn_back.pack(pady=10)

def confirm_exit():
    """Подтверждение выхода из приложения."""
    if messagebox.askokcancel("Выход", "Вы действительно хотите выйти?"):
        root.destroy()

root = tk.Tk()
root.title("Заметки")
root.geometry("450x450")
root.resizable(False, False)

show_start_screen()

root.mainloop()
