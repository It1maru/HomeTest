import tkinter as tk
from tkinter import messagebox, simpledialog, scrolledtext

class NoteApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Заметки с ответвлениями")
        self.geometry("700x500")
        self.resizable(False, False)

        # Данные: две ветки заметок
        self.branches = {
            "Ветка 1": [],
            "Ветка 2": []
        }
        self.current_branch = None
        self.current_note_index = None

        self.create_widgets()
        self.show_start_page()

    def create_widgets(self):
        # Верхняя панель с кнопками переключения веток
        self.top_frame = tk.Frame(self)
        self.top_frame.pack(side=tk.TOP, fill=tk.X)

        self.btn_branch1 = tk.Button(self.top_frame, text="Ветка 1", command=lambda: self.switch_branch("Ветка 1"))
        self.btn_branch1.pack(side=tk.LEFT, padx=5, pady=5)

        self.btn_branch2 = tk.Button(self.top_frame, text="Ветка 2", command=lambda: self.switch_branch("Ветка 2"))
        self.btn_branch2.pack(side=tk.LEFT, padx=5, pady=5)

        self.btn_start = tk.Button(self.top_frame, text="Начальная страница", command=self.show_start_page)
        self.btn_start.pack(side=tk.LEFT, padx=5, pady=5)

        # Левая панель - список заметок
        self.left_frame = tk.Frame(self, width=200)
        self.left_frame.pack(side=tk.LEFT, fill=tk.Y, padx=5, pady=5)

        self.lbl_notes = tk.Label(self.left_frame, text="Заметки")
        self.lbl_notes.pack()

        self.listbox_notes = tk.Listbox(self.left_frame, width=30, height=25)
        self.listbox_notes.pack(pady=5)
        self.listbox_notes.bind("<<ListboxSelect>>", self.on_note_select)

        self.btn_add_note = tk.Button(self.left_frame, text="Добавить заметку", command=self.add_note)
        self.btn_add_note.pack(fill=tk.X, pady=2)

        self.btn_delete_note = tk.Button(self.left_frame, text="Удалить заметку", command=self.delete_note)
        self.btn_delete_note.pack(fill=tk.X, pady=2)

        # Правая панель - текст заметки
        self.right_frame = tk.Frame(self)
        self.right_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=5, pady=5)

        self.lbl_note_title = tk.Label(self.right_frame, text="Выберите заметку", font=("Arial", 14))
        self.lbl_note_title.pack()

        self.text_note = scrolledtext.ScrolledText(self.right_frame, wrap=tk.WORD, state=tk.DISABLED)
        self.text_note.pack(fill=tk.BOTH, expand=True)

        self.btn_save_note = tk.Button(self.right_frame, text="Сохранить изменения", command=self.save_note, state=tk.DISABLED)
        self.btn_save_note.pack(pady=5)

    def show_start_page(self):
        self.current_branch = None
        self.current_note_index = None
        self.listbox_notes.delete(0, tk.END)
        self.lbl_note_title.config(text="Добро пожаловать в приложение Заметки!")
        self.text_note.config(state=tk.NORMAL)
        self.text_note.delete(1.0, tk.END)
        self.text_note.insert(tk.END, "Выберите ветку заметок слева, чтобы начать работу.\n\n"
                                      "Ветка 1 и Ветка 2 — это два независимых набора заметок.\n"
                                      "Вы можете создавать, редактировать и удалять заметки в каждой ветке.")
        self.text_note.config(state=tk.DISABLED)
        self.btn_save_note.config(state=tk.DISABLED)

    def switch_branch(self, branch_name):
        self.current_branch = branch_name
        self.current_note_index = None
        self.lbl_note_title.config(text=f"Ветка: {branch_name}")
        self.refresh_notes_list()
        self.text_note.config(state=tk.NORMAL)
        self.text_note.delete(1.0, tk.END)
        self.text_note.config(state=tk.DISABLED)
        self.btn_save_note.config(state=tk.DISABLED)

    def refresh_notes_list(self):
        self.listbox_notes.delete(0, tk.END)
        notes = self.branches[self.current_branch]
        for i, note in enumerate(notes):
            title = note['title'] if note['title'] else f"Заметка {i+1}"
            self.listbox_notes.insert(tk.END, title)

    def add_note(self):
        if not self.current_branch:
            messagebox.showwarning("Внимание", "Сначала выберите ветку заметок.")
            return
        title = simpledialog.askstring("Новая заметка", "Введите заголовок заметки:")
        if title is None:
            return
        note = {"title": title, "content": ""}
        self.branches[self.current_branch].append(note)
        self.refresh_notes_list()
        self.listbox_notes.select_set(tk.END)
        self.on_note_select()

    def delete_note(self):
        if self.current_branch is None or self.current_note_index is None:
            messagebox.showwarning("Внимание", "Выберите заметку для удаления.")
            return
        confirm = messagebox.askyesno("Удалить", "Вы уверены, что хотите удалить эту заметку?")
        if confirm:
            del self.branches[self.current_branch][self.current_note_index]
            self.current_note_index = None
            self.refresh_notes_list()
            self.text_note.config(state=tk.NORMAL)
            self.text_note.delete(1.0, tk.END)
            self.text_note.config(state=tk.DISABLED)
            self.lbl_note_title.config(text=f"Ветка: {self.current_branch}")
            self.btn_save_note.config(state=tk.DISABLED)

    def on_note_select(self, event=None):
        if self.current_branch is None:
            return
        selection = self.listbox_notes.curselection()
        if not selection:
            return
        index = selection[0]
        self.current_note_index = index
        note = self.branches[self.current_branch][index]
        self.lbl_note_title.config(text=note['title'])
        self.text_note.config(state=tk.NORMAL)
        self.text_note.delete(1.0, tk.END)
        self.text_note.insert(tk.END, note['content'])
        self.btn_save_note.config(state=tk.NORMAL)

    def save_note(self):
        if self.current_branch is None or self.current_note_index is None:
            return
        content = self.text_note.get(1.0, tk.END).rstrip()
        self.branches[self.current_branch][self.current_note_index]['content'] = content
        messagebox.showinfo("Сохранено", "Изменения сохранены.")

if __name__ == "__main__":
    app = NoteApp()
    app.mainloop()