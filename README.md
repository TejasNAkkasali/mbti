import tkinter as tk
from tkinter import messagebox
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# MBTI data dictionary (sample, add more types as needed)
mbti_data = {
    "ENTJ": {
        "description": "The Commander – Strong-willed, confident, and strategic.",
        "strengths": ["Leadership", "Strategic", "Energetic", "Confident"],
        "weaknesses": ["Dominant", "Insensitive", "Perfectionist"]
    }
    # Add other MBTI types here...
}

# 12 MBTI questions
questions = [
    # I vs E
    "I enjoy solitary activities more than group events.",
    "I prefer deep one-on-one conversations over group discussions.",
    "I recharge best when I'm alone.",
    # S vs N
    "I trust experience more than theoretical ideas.",
    "I focus more on present details than future possibilities.",
    "I prefer realistic plans to imaginative dreams.",
    # T vs F
    "I value logic over emotions when making decisions.",
    "I prefer honesty over tact, even if it hurts feelings.",
    "I often analyze pros and cons before acting.",
    # J vs P
    "I like planning things well in advance.",
    "I feel uncomfortable when things are unorganized.",
    "I prefer structured routines over spontaneous activities."
]

class MBTITestApp:
    def __init__(self, root):
        self.root = root
        self.root.title("MBTI Personality Test")
        self.root.configure(bg="#e8f0fe")
        self.scales = []

        tk.Label(
            root, text="MBTI Personality Test", font=("Helvetica", 22, "bold"),
            fg="#1a237e", bg="#e8f0fe"
        ).pack(pady=20)

        canvas = tk.Canvas(root, bg="#e8f0fe", highlightthickness=0)
        frame = tk.Frame(canvas, bg="#e8f0fe")
        scrollbar = tk.Scrollbar(root, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scrollbar.set)

        scrollbar.pack(side="right", fill="y")
        canvas.pack(side="left", fill="both", expand=True)
        canvas.create_window((0, 0), window=frame, anchor='nw')
        frame.bind("<Configure>", lambda e: canvas.configure(scrollregion=canvas.bbox("all")))

        for i, q in enumerate(questions):
            tk.Label(
                frame, text=q, font=("Arial", 11), bg="#e8f0fe", wraplength=500, justify="left"
            ).grid(row=i, column=0, sticky="w", padx=10, pady=5)
            scale = tk.Scale(
                frame, from_=1, to=5, orient=tk.HORIZONTAL, bg="#e8f0fe", highlightthickness=0,
                troughcolor="#c5cae9", fg="#1a237e", length=250
            )
            scale.grid(row=i, column=1, padx=10)
            self.scales.append(scale)

        self.submit_btn = tk.Button(
            root, text="Submit", font=("Arial", 13, "bold"), bg="#3f51b5", fg="white",
            activebackground="#303f9f", activeforeground="white", padx=20, pady=10,
            bd=0, relief="flat", command=self.submit
        )
        self.submit_btn.pack(pady=20)
        self.submit_btn.bind("<Enter>", lambda e: self.submit_btn.config(bg='#303f9f'))
        self.submit_btn.bind("<Leave>", lambda e: self.submit_btn.config(bg='#3f51b5'))

    def submit(self):
        responses = [scale.get() for scale in self.scales]
        if len(responses) < 12:
            messagebox.showerror("Incomplete", "Please answer all questions.")
            return

        mbti_type = self.analyze_mbti(np.array(responses))
        messagebox.showinfo("Your MBTI Type", f"Your estimated MBTI type: {mbti_type}")
        self.visualize_results(mbti_type)
        self.generate_report(mbti_type)

    def analyze_mbti(self, responses):
        traits = {
            'I/E': np.mean(responses[0:3]),
            'S/N': np.mean(responses[3:6]),
            'T/F': np.mean(responses[6:9]),
            'J/P': np.mean(responses[9:12])
        }
        mbti = ''.join(trait[1] if score >= 3 else trait[0] for trait, score in traits.items())
        return mbti

    def visualize_results(self, mbti_type):
        plt.figure(figsize=(5, 3))
        categories = list(mbti_type)
        values = [1] * 4
        plt.bar(categories, values, color=['#4CAF50', '#2196F3', '#FF5722', '#9C27B0'])
        plt.xlabel("MBTI Traits")
        plt.ylabel("Score")
        plt.title(f"MBTI Type: {mbti_type}")
        plt.tight_layout()
        plt.show()

    def generate_report(self, mbti_type):
        if mbti_type in mbti_data:
            data = mbti_data[mbti_type]
            df = pd.DataFrame({
                "MBTI Type": [mbti_type],
                "Description": [data['description']],
                "Strengths": [', '.join(data['strengths'])],
                "Weaknesses": [', '.join(data['weaknesses'])]
            })
            df.to_csv("MBTI_Report.csv", index=False)
            messagebox.showinfo("Report Generated", "Report saved as MBTI_Report.csv")
        else:
            messagebox.showwarning("Unknown Type", "No data available for this MBTI type.")

# Run app
if __name__ == "__main__":
    root = tk.Tk()
    root.geometry("700x600")
    app = MBTITestApp(root)
    root.mainloop()
    
