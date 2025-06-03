import os
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
import tkinter as tk
from tkinter import ttk, messagebox, filedialog
from matplotlib.backends.backend_tkagg
import FigureCanvasTkAgg


DATA_FILE = "ad_data_realistic.csv"
EXCEL_FILE = "ad_data_realistic.xlsx"
CHART_DIR = "charts"

class AdApp:
    def __init__(self, root):
        self.root = root
        self.root.title("\U0001F4CA Анализ рекламных затрат и продаж")
        self.root.geometry("1000x700")
        self.style = ttk.Style()
        self.style.theme_use("clam")
        self.style.configure("Treeview.Heading", font=('Segoe UI', 10, 'bold'))
        self.style.configure("TNotebook.Tab", font=('Segoe UI', 10, 'bold'))

        self.df = pd.DataFrame(columns=['Period', 'TV', 'Internet', 'Outdoor', 'Sales'])

        self.notebook = ttk.Notebook(self.root)
        self.notebook.pack(fill="both", expand=True)

        self.data_frame = ttk.Frame(self.notebook)
        self.analysis_frame = ttk.Frame(self.notebook)
        self.visual_frame = ttk.Frame(self.notebook)

        self.notebook.add(self.data_frame, text="\U0001F4C1 Данные")
        self.notebook.add(self.analysis_frame,
text="\U0001F4CA Анализ и прогноз")
        self.notebook.add(self.visual_frame, text="\U0001F4C8 Визуализация")

        self.build_data_tab()
        self.build_analysis_tab()
        self.build_visual_tab()

    def load_file(self):
        file_path =
filedialog.askopenfilename(
            title="Выберите CSV или Excel файл",
            filetypes=[("CSV files", "*.csv"), ("Excelfiles", "*.xlsx *.xls")]
        )
        if not file_path:
            return
        try:
            if file_path.lower().endswith(".csv"):
                df_new =
pd.read_csv(file_path)
            else:
                df_new = pd.read_excel(file_path)
            expected_cols = ['Period', 'TV', 'Internet', 'Outdoor', 'Sales']
            if not all(col in df_new.columns for col in expected_cols):
                messagebox.showerror("Ошибка", f"Файл должен содержать колонки: {expected_cols}")
                return
            self.df = df_new[expected_cols].drop_duplicates(subset=['Period']).reset_index(drop=True)
            self.update_tree()
            messagebox.showinfo("Успех", f"Данные загружены из {os.path.basename(file_path)}")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось загрузить файл:\n{e}")


    def save_data(self):
        try:
            self.df.to_csv(DATA_FILE, index=False)
            self.df.to_excel(EXCEL_FILE, index=False)
            messagebox.showinfo("Сохранение", "Данные успешно сохранены!")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Ошибка сохранения данных:\n{e}")

    def build_data_tab(self):
        frame = self.data_frame

        load_btn = ttk.Button(frame, text="\U0001F4C2 Загрузить CSV/Excel", command=self.load_file)
        load_btn.pack(pady=5)

        self.tree = ttk.Treeview(frame, columns=['Period', 'TV', 'Internet', 'Outdoor', 'Sales'], show='headings', height=20)
        for col in ['Period', 'TV', 'Internet', 'Outdoor', 'Sales']:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=120)
        self.tree.pack(fill="both", expand=True, padx=10, pady=10)

       
btn_frame = ttk.Frame(frame)
        btn_frame.pack(pady=5)

        ttk.Button(btn_frame, text="\u2795 Добавить", command=self.open_add_window).grid(row=0, column=0, padx=5)
        ttk.Button(btn_frame, text="\u270F\uFE0F Редактировать", command=self.open_edit_window).grid(row=0, column=1, padx=5)
        ttk.Button(btn_frame, text="\u274C Удалить", command=self.delete_entry).grid(row=0, column=2, padx=5)
        ttk.Button(btn_frame, text="\U0001F4BE Сохранить", command=self.save_data).grid(row=0, column=3, padx=5)

    def open_add_window(self):
        self.open_edit_window(is_new=True)

    def open_edit_window(self, is_new=False):
        selected = self.tree.selection()
        if not is_new and not
not is_new and not selected:
            messagebox.showwarning("Выбор", "Выберите запись для редактирования.")
            return

        top =
tk.Toplevel(self.root)
        top.title("Добавить" if is_new else "Редактировать")
        top.geometry("300x300")

        labels = ['Period', 'TV', 'Internet', 'Outdoor', 'Sales']
        entries = {}

        for i, label in enumerate(labels):
            ttk.Label(top, text=label).grid(row=i, column=0, padx=5, pady=5)
            entry = ttk.Entry(top)
            entry.grid(row=i, column=1, padx=5, pady=5)
            entries[label] = entry

        if not is_new:
            values = self.tree.item(selected[0], 'values')
            for i, label in enumerate(labels):
                entries[label].insert(0, values[i])

        def save():
            try:
                period = entries['Period'].get()
                tv = float(entries['TV'].get())
                internet = float(entries['Internet'].get())
                outdoor = float(entries['Outdoor'].get())
                sales = float(entries['Sales'].get())
                if is_new:
                    if period in self.df['Period'].values:
                       
messagebox.showerror("Ошибка", "Период уже существует.")
                        return
                    self.df.loc[len(self.df)] = [period, tv, internet, outdoor, sales]
                else:
                    idx = self.df[self.df['Period'] == values[0]].index[0]
                    self.df.loc[idx] = [period, tv, internet, outdoor, sales]
                self.update_tree()
                top.destroy()
            except Exception:
                messagebox.showerror("Ошибка", "Введите корректные числовые значения.")

        ttk.Button(top, text="Сохранить", command=save).grid(row=len(labels), columnspan=2, pady=10)

    def update_tree(self):
        for row in self.tree.get_children():
            self.tree.delete(row)
        for _, row in self.df.iterrows():
            self.tree.insert("", "end", values=list(row))

    def delete_entry(self):
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Выбор", "Выберите запись для удаления.")
            return
        values = self.tree.item(selected[0], 'values')
        self.df = self.df[self.df['Period'] != values[0]]
        self.update_tree()

    def build_analysis_tab(self):
        frame = self.analysis_frame

        input_frame =
ttk.LabelFrame(frame, text="Ввод рекламных затрат для прогноза (в тыс. руб.)", padding=10)
        input_frame.pack(fill="x", padx=10, pady=10)

        # Размещение по строкам, а не в одну строку
        ttk.Label(input_frame,
text="Затраты на ТВ рекламу:").grid(row=0, column=0, sticky='e', padx=5, pady=5)
        self.tv_entry = ttk.Entry(input_frame, width=20)
        self.tv_entry.grid(row=0, column=1, sticky='w', padx=5, pady=5)

        ttk.Label(input_frame, text="Затраты на рекламу в интернете:").grid(row=1, column=0, sticky='e', padx=5, pady=5)
        self.internet_entry = ttk.Entry(input_frame, width=20)
        self.internet_entry.grid(row=1, column=1, sticky='w', padx=5, pady=5)

        ttk.Label(input_frame, text="Затраты на наружную рекламу:").grid(row=2, column=0, sticky='e', padx=5, pady=5)
        self.outdoor_entry = ttk.Entry(input_frame, width=20)
        self.outdoor_entry.grid(row=2, column=1, sticky='w', padx=5, pady=5)

        # Кнопка запуска анализа
        analyze_btn =
ttk.Button(input_frame, text="\U0001F50D Выполнить анализ и прогноз", command=self.analyze)
        analyze_btn.grid(row=3, columnspan=2, pady=10)

        # Вывод результатов анализа
        self.analysis_output = tk.Text(frame, height=20, wrap="word", font=("Segoe UI", 10))
        self.analysis_output.pack(fill="both", expand=False, padx=10, pady=(0, 10))

        # График прогноза
        chart_frame =
ttk.LabelFrame(frame, text="График прогноза объёма продаж", padding=10)
        chart_frame.pack(fill="both", expand=True, padx=10, pady=10)
        self.pred_fig = plt.Figure(figsize=(7, 2.5), dpi=100)

self.pred_canvas = FigureCanvasTkAgg(self.pred_fig, master=chart_frame)
        self.pred_canvas.get_tk_widget().pack(fill="both", expand=True)

    def analyze(self):
        self.analysis_output.delete("1.0", tk.END)

        if self.df.empty:
            self.analysis_output.insert(tk.END, "❌ Нет данных для анализа.\n")
            return

        try:
            X = self.df[['TV', 'Internet', 'Outdoor']]
            y = self.df['Sales']
            model = LinearRegression()
            model.fit(X, y)
            r2 = model.score(X, y)
            correlations = self.df.corr(numeric_only=True)['Sales'].drop('Sales')

            self.analysis_output.insert(tk.END, "📌 Корреляция (влияние на продажи):\n")
            for name, corr in correlations.items():
                self.analysis_output.insert(tk.END,
                                           
f"- {name}: {corr:.2f} ({'прямая' if corr > 0 else 'обратная'} связь)\n")
            self.analysis_output.insert(tk.END, "\n")

            self.analysis_output.insert(tk.END, f"📊 Коэффициент детерминации (R²): {r2:.2f}\n")
            if r2 < 0.5:
                self.analysis_output.insert(tk.END, "🔎 Модель плохо объясняет данные. Возможны скрытые факторы.\n")
            elif r2 < 0.8:
                self.analysis_output.insert(tk.END, "ℹ️ Модель средней точности. Можно использовать с осторожностью.\n")
            else:
                self.analysis_output.insert(tk.END, "✅ Модель достаточно точна для прогнозов.\n")
            self.analysis_output.insert(tk.END, "\n")

            self.analysis_output.insert(tk.END, "🧮 Коэффициенты регрессии:\n")
            for name, coef in zip(X.columns, model.coef_):
                direction = "увеличиваются" if coef > 0 else "уменьшаются"
                self.analysis_output.insert(
                    tk.END, f"- При увеличении {name} на 1 тыс. руб. продажи {direction} на {abs(coef):.2f} тыс. руб.\n"
                )
            self.analysis_output.insert(tk.END,
                                        f"- Базовый уровень продаж (без затрат):
{model.intercept_:.2f} тыс. руб.\n\n")

            # Прогноз
            tv = float(self.tv_entry.get())
            internet = float(self.internet_entry.get())
            outdoor = float(self.outdoor_entry.get())

            pred = model.predict([[tv, internet, outdoor]])[0]

            self.analysis_output.insert(tk.END, f"🔮 Прогнозируемый объём продаж: {pred:.2f} тыс. руб.\n\n")

            # Вывод
            max_corr = correlations.abs().idxmax()
            self.analysis_output.insert(tk.END, "🧾 Вывод:\n")
            self.analysis_output.insert(tk.END, f"- Наибольшее влияние на продажи оказывает
реклама в: {max_corr}.\n")
            self.analysis_output.insert(tk.END, "- Используйте наиболее эффективный канал для увеличенияпродаж.\n")


            self.plot_prediction(tv, internet, outdoor, pred)

        except ValueError:
            self.analysis_output.insert(tk.END, "⚠️ Введите корректные числовые значения затрат.\n")
        except Exception as e:
            self.analysis_output.insert(tk.END, f"❌ Ошибка при анализе: {e}\n")

    def plot_prediction(self, tv, internet, outdoor, prediction):
        self.pred_fig.clf()
        ax = self.pred_fig.add_subplot(111)

        bars = ['TV', 'Internet', 'Outdoor', 'Прогноз']
        values = [tv, internet, outdoor,
prediction]

        colors = ['#77c7f2', '#a6e3a1', '#cba0e3', '#ffb347']

        bars_container = ax.bar(bars,
values, color=colors, alpha=0.85, edgecolor='gray', linewidth=1.2)

        # Добавление текстов над столбцами
        for bar in bars_container:
            height = bar.get_height()
            ax.annotate(f"{height:.1f}",
                        xy=(bar.get_x() + bar.get_width() / 2, height),
                        xytext=(0, 5),
                        textcoords="offset points",
                        ha='center', va='bottom',
                        fontsize=10, fontweight='bold', color='black')

        ax.set_title("Входные затраты и прогноз
продаж", fontsize=13,

fontweight='bold')
        ax.set_ylabel("Тысячи рублей", fontsize=11)
        ax.set_ylim(0, max(values) * 1.25)  # Добавить немного пространства сверху
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.grid(axis='y', linestyle='--', alpha=0.4)
        ax.tick_params(axis='x', labelsize=10)
        ax.tick_params(axis='y', labelsize=10)

        self.pred_canvas.draw()

    def build_visual_tab(self):
        frame = self.visual_frame
        ttk.Button(frame, text="\U0001F4C9 Построить графики", command=self.visualize).pack(pady=10)
        self.canvas_frame = ttk.Frame(frame)
        self.canvas_frame.pack(fill="both", expand=True)

    def visualize(self):
        for widget in self.canvas_frame.winfo_children():
            widget.destroy()

        if self.df.empty:
            messagebox.showwarning("Нет данных", "Добавьте данные для визуализации.")
            return

        self.ensure_chart_dir()

        fig, axes = plt.subplots(2, 2, figsize=(10, 7))

        sns.regplot(x='TV', y='Sales', data=self.df, ax=axes[0, 0], scatter_kws={'s':50}, line_kws={'color':'orange'})
        axes[0, 0].set_title("TV vs Sales")

        sns.regplot(x='Internet', y='Sales', data=self.df, ax=axes[0, 1], scatter_kws={'s':50}, line_kws={'color':'green'})
        axes[0, 1].set_title("Internet vs Sales")

        sns.regplot(x='Outdoor', y='Sales', data=self.df, ax=axes[1, 0], scatter_kws={'s':50}, line_kws={'color':'purple'})
        axes[1, 0].set_title("Outdoor vs Sales")

        sns.heatmap(self.df[['TV', 'Internet', 'Outdoor', 'Sales']].corr(), annot=True, cmap="coolwarm", ax=axes[1, 1])
        axes[1, 1].set_title("Корреляционная матрица")

        plt.tight_layout()
       
fig.savefig(os.path.join(CHART_DIR, "visual_analysis.png"))

        canvas = FigureCanvasTkAgg(fig, master=self.canvas_frame)
        canvas.draw()
        canvas.get_tk_widget().pack(fill="both", expand=True)

    def ensure_chart_dir(self):
        if not os.path.exists(CHART_DIR):
            os.makedirs(CHART_DIR)

if name == "__main__":
    root = tk.Tk()
    app = AdApp(root)
    root.mainloop()
