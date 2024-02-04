import tkinter as tk
from tkinter import ttk
import math

class TimeCalculatorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("YouTube Debug Time Calculator")

        self.header_frame = tk.Frame(root)
        self.header_frame.pack(side="top", fill="both", expand=False)

        self.middle_canvas = tk.Canvas(root, highlightthickness=0)
        self.middle_canvas.pack(side="top", fill="both", expand=True)

        self.footer_frame = tk.Frame(root)
        self.footer_frame.pack(side="top", fill="both", expand=False)

        self.scrollbar = ttk.Scrollbar(self.middle_canvas, orient="vertical", command=self.middle_canvas.yview)
        self.scrollbar.pack(side="right", fill="y")

        self.middle_canvas.configure(yscrollcommand=self.scrollbar.set)

        self.root.bind_all("<MouseWheel>", self.on_mousewheel)

        self.middle_canvas.bind("<Configure>", self.configure_scroll_region)

        total_height = 800
        header_height = 50  
        footer_height = 100 
        middle_height = total_height - header_height - footer_height

        self.top_label = tk.Label(self.header_frame, text="YouTube Debug Time Calculator", font=("Arial", 16))
        self.top_label.pack(fill="both", expand=True)

        self.segments = []
        self.segment_counter = 1

        self.create_segment()

        self.bottom_label = tk.Label(self.footer_frame, text="Total: 00:00:00.000 - from 1 segment", font=("Arial", 12))
        self.bottom_label.grid(row=0, column=0, padx=10, pady=5, sticky="w")

        self.frame_rate_var = tk.StringVar()
        self.frame_rate_label = tk.Label(self.footer_frame, text="Frame Rate (fps):", font=("Arial", 12))
        self.frame_rate_label.grid(row=1, column=0, padx=10, pady=5, sticky="w")

        self.frame_rate_entry = tk.Entry(self.footer_frame, font=("Arial", 12), textvariable=self.frame_rate_var, width=5)
        self.frame_rate_entry.grid(row=1, column=1, padx=10, pady=5, sticky="w")

        self.frame_rate_var.trace_add("write", self.update_total_time)

        self.caution_label = tk.Label(self.footer_frame, text="", font=("Arial", 12), fg="red")
        self.caution_label.grid(row=2, column=0, columnspan=2, padx=10, pady=5, sticky="w")

        self.root.wm_attributes("-topmost", 1)  

    def configure_scroll_region(self, event):
        self.middle_canvas.update_idletasks()
        self.middle_canvas.configure(scrollregion=self.middle_canvas.bbox("all"))

    def on_mousewheel(self, event):
        self.middle_canvas.yview_scroll(int(-1*(event.delta/120)), "units")

    def create_segment(self):
        segment_frame = ttk.LabelFrame(self.middle_canvas, text=str(self.segment_counter) + ".", padding=10)
        self.middle_canvas.create_window((0, len(self.segments) * 180), window=segment_frame, anchor="nw")

        start_debug_info_var = tk.StringVar()
        end_debug_info_var = tk.StringVar()
        segment_time_var = tk.StringVar()

        start_debug_info_label = tk.Label(segment_frame, text="Enter Start Time:", font=("Arial", 12))
        end_debug_info_label = tk.Label(segment_frame, text="Enter End Time:", font=("Arial", 12))
        segment_time_label = tk.Label(segment_frame, textvariable=segment_time_var, font=("Arial", 12))

        start_debug_info_entry = tk.Entry(segment_frame, font=("Arial", 18), justify="center", textvariable=start_debug_info_var, width=12)
        end_debug_info_entry = tk.Entry(segment_frame, font=("Arial", 18), justify="center", textvariable=end_debug_info_var, width=12)

        start_debug_info_label.grid(row=0, column=0, sticky="nsew")
        start_debug_info_entry.grid(row=0, column=1, sticky="nsew")
        end_debug_info_label.grid(row=1, column=0, sticky="nsew")
        end_debug_info_entry.grid(row=1, column=1, sticky="nsew")
        segment_time_label.grid(row=2, column=0, columnspan=3)

        delete_button = ttk.Button(segment_frame, text="-", width=2, command=lambda: self.delete_segment(segment_frame))
        delete_button.grid(row=0, column=2, padx=(5, 0))

        add_button = ttk.Button(segment_frame, text="+", command=self.add_segment)
        add_button.grid(row=3, column=0, columnspan=3, sticky="nsew")

        start_debug_info_var.trace_add("write", self.update_total_time)
        end_debug_info_var.trace_add("write", self.update_total_time)

        self.segments.append((start_debug_info_var, end_debug_info_var, segment_frame, segment_time_var))

        self.segment_counter += 1
        self.configure_scroll_region(None)

    def delete_segment(self, segment_frame):
        for i, segment_tuple in enumerate(self.segments):
            if segment_tuple[2] == segment_frame:
                self.segments.remove(segment_tuple)
                segment_frame.destroy()
                self.update_total_time()
                
                for j, (_, _, frame, _) in enumerate(self.segments):
                    frame.config(text=f"{j + 1}.")
                break

    def add_segment(self):
        prev_segment_frame = self.segments[-1][2]  
        self.create_segment()
        prev_segment_frame.tkraise()  
        self.update_total_time()

    def parse_vct_value(self, debug_info, frame_rate):
        start_index = debug_info.find('"vct": "') + len('"vct": "')
        end_index = debug_info.find('"', start_index)
        vct_value = float(debug_info[start_index:end_index])

        milliseconds = (vct_value - (vct_value % (1 / frame_rate))) * 1000

        return milliseconds / 1000

    def format_time(self, total_time):
        hours = int(total_time / 3600)
        minutes = int((total_time % 3600) / 60)
        seconds = int(total_time % 60)
        milliseconds = int((total_time % 1) * 1000)
        if hours > 0:
            return f"{hours:02d}:{minutes:02d}:{seconds:02d}:{milliseconds:03d}"
        else:
            return f"{minutes:02d}:{seconds:02d}:{milliseconds:03d}"

    def update_total_time(self, *args):
        total_time = 0.0
        empty_segments = 0

        frame_rate = self.frame_rate_var.get()
        if frame_rate:
            frame_rate = float(frame_rate)

        for segment_tuple in self.segments:
            start_debug_info = segment_tuple[0].get()
            end_debug_info = segment_tuple[1].get()

            if start_debug_info and end_debug_info:
                start_time = self.parse_vct_value(start_debug_info, frame_rate)
                end_time = self.parse_vct_value(end_debug_info, frame_rate)

                segment_time = end_time - start_time
                segment_tuple[3].set(f"Segment Time: {self.format_time(segment_time)}")

                total_time += segment_time
            else:
                empty_segments += 1

        self.bottom_label.config(text=f"Total: {self.format_time(total_time)} - from {len(self.segments)} segments")
        
        if empty_segments > 0:
            self.caution_label.config(text=f"Caution - {empty_segments} segments left empty")
        else:
            self.caution_label.config(text="")

if __name__ == "__main__":
    root = tk.Tk()
    app = TimeCalculatorApp(root)
    root.geometry("400x800")
    root.wm_attributes("-topmost", 1) 
    root.mainloop()
