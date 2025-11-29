import customtkinter as ctk
from tkinter import messagebox, Toplevel
import google.generativeai as genai


genai.configure(api_key="YOUR API HERE")

# Load model
model = genai.GenerativeModel("gemini-2.0-flash")


ctk.set_appearance_mode("light")
ctk.set_default_color_theme("blue")

app = ctk.CTk()
app.title("Study Mode Assistant")
app.geometry("1000x720")

# Store History
chat_history = []

top_section = ctk.CTkFrame(app)
top_section.pack(fill="x", padx=10, pady=10)

# Question Frame
question_frame = ctk.CTkFrame(top_section)
question_frame.grid(row=0, column=0, padx=10, pady=5, sticky="w")

question_entry = ctk.CTkEntry(
    question_frame,
    placeholder_text="Type your question here...",
    width=500,
    height=40
)
question_entry.pack(side="left", fill="x", expand=True)

send_btn = ctk.CTkButton(
    question_frame,
    text="➤",
    width=40,
    height=40,
    fg_color="#1f6aa5",
    hover_color="#145e96",
    command=lambda: send_question()
)
send_btn.pack(side="right", padx=5)

# Subject Menu
subject_var = ctk.StringVar(value="Mathematics")
subject_menu = ctk.CTkOptionMenu(
    top_section,
    values=["Mathematics", "Physics", "Biology", "Chemistry"],
    variable=subject_var,
    width=180
)
subject_menu.grid(row=0, column=1, padx=10)

# Task Menu
task_var = ctk.StringVar(value="Solve Problem")
task_menu = ctk.CTkOptionMenu(
    top_section,
    values=["Solve Problem", "Explain Concept", "Summarize Topic", "Give Example"],
    variable=task_var,
    width=180
)
task_menu.grid(row=0, column=2, padx=10)

# Theme Menu
def toggle_theme(choice):
    ctk.set_appearance_mode(choice)

theme_menu = ctk.CTkOptionMenu(
    top_section,
    values=["Light", "Dark", "System"],
    command=toggle_theme,
    width=120
)
theme_menu.grid(row=0, column=3, padx=10)



#  BUTTON ROW
button_row = ctk.CTkFrame(app)
button_row.pack(fill="x", padx=10, pady=5)

def clear_answer():
    answer_box.configure(state="normal")
    answer_box.delete("1.0", "end")
    answer_box.insert("1.0", "Answer will appear here...")
    answer_box.configure(state="disabled")

clear_btn = ctk.CTkButton(
    button_row,
    text="Clear Answer",
    fg_color="red",
    width=120,
    command=clear_answer
)
clear_btn.grid(row=0, column=0, padx=10)

save_btn = ctk.CTkButton(button_row, text="Save Answer", width=120, state="disabled")
save_btn.grid(row=0, column=1, padx=10)

def open_history_window():
    history_win = Toplevel(app)
    history_win.title("Chat History")
    history_win.geometry("500x600")

    history_box = ctk.CTkTextbox(history_win, font=("Arial", 14))
    history_box.pack(fill="both", expand=True, padx=10, pady=10)

    history_box.insert("end", "===== Chat History =====\n\n")

    for i, (q, a) in enumerate(chat_history, start=1):
        history_box.insert("end", f"{i}. Question:\n{q}\n\n")
        history_box.insert("end", f"Answer:\n{a}\n")
        history_box.insert("end", "-"*40 + "\n")

    history_box.configure(state="disabled")

chat_history_btn = ctk.CTkButton(
    button_row,
    text="Chat History",
    width=120,
    fg_color="#3b8ed0",
    command=open_history_window
)
chat_history_btn.grid(row=0, column=2, padx=10)


#  ANSWER BOX
answer_frame = ctk.CTkFrame(app)
answer_frame.pack(fill="both", expand=True, padx=10, pady=10)

answer_box = ctk.CTkTextbox(answer_frame, height=430, font=("Arial", 15))
answer_box.pack(fill="both", expand=True)

answer_box.configure(state="normal")
answer_box.insert("1.0", "Answer will appear here...")
answer_box.configure(state="disabled")

def block_user(event): return "break"
answer_box.bind("<Key>", block_user)
answer_box.bind("<Button-1>", block_user)
answer_box.bind("<FocusIn>", lambda e: question_entry.focus())


#  GEMINI API CALL
def send_question(event=None):
    question = question_entry.get().strip()
    if not question:
        messagebox.showwarning("Warning", "Enter a question first.")
        return

    subject = subject_var.get()
    task = task_var.get()

    prompt = (
        f"You are an expert {subject} tutor specializing in {task}.\n\n"
        f"User question: {question}"
    )

    # Call Gemini API
    try:
        response = model.generate_content(prompt)
        answer = response.text or "No response generated."

    except Exception as e:
        answer = f"Error: {str(e)}"

    # Show answer
    answer_box.configure(state="normal")
    answer_box.delete("1.0", "end")
    answer_box.insert("end", answer)
    answer_box.configure(state="disabled")

    # Save to history
    chat_history.append((question, answer))

    # Enable save button
    def save_answer():
        with open("saved_answers.txt", "a", encoding="utf-8") as f:
            f.write(f"SUBJECT: {subject}\nTASK: {task}\nQ: {question}\nA: {answer}\n\n")
        messagebox.showinfo("Saved", "Answer saved successfully!")

    save_btn.configure(state="normal", command=save_answer)
    question_entry.delete(0, "end")


# Bind Enter
question_entry.bind("<Return>", send_question)


#  RUN APP
app.mainloop()
