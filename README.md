# FerNotes

My first project

import tkinter as tk
from tkinter import colorchooser, filedialog, simpledialog
import json
from PIL import Image, ImageGrab

WIDTH = 1000
HEIGHT = 700
BG_COLOR = "#fffffe"

current_color = "#3d3d3d"
brush_size = 5
eraser_mode = False
text_mode = False

root = tk.Tk()
root.title("FerNotes ✨")
root.geometry(f"{WIDTH}x{HEIGHT}")
root.configure(bg="#f4f0ff")
root.resizable(True, True)

last_x = None
last_y = None

undo_stack = []
redo_stack = []
current_stroke = []

def set_color(color):
global current_color
global eraser_mode
global text_mode

    current_color = color
    eraser_mode = False
    text_mode = False

def choose_color():
global current_color
global eraser_mode
global text_mode

    color = colorchooser.askcolor()[1]

    if color:
        current_color = color
        eraser_mode = False
        text_mode = False

def toggle_eraser():
global eraser_mode
global text_mode

    eraser_mode = not eraser_mode
    text_mode = False

def toggle_text():
global text_mode
global eraser_mode

    text_mode = not text_mode
    eraser_mode = False

def clear_canvas(event=None):

    canvas.delete("all")

    draw_lines()

    undo_stack.clear()
    redo_stack.clear()

def undo(event=None):

    if undo_stack:

        stroke = undo_stack.pop()

        for item in stroke:
            canvas.itemconfigure(item, state="hidden")

        redo_stack.append(stroke)

def redo(event=None):

    if redo_stack:

        stroke = redo_stack.pop()

        for item in stroke:
            canvas.itemconfigure(item, state="normal")

        undo_stack.append(stroke)

def update_brush_size(value):
global brush_size

    brush_size = int(value)

def start_draw(event):
global last_x
global last_y
global current_stroke

    current_stroke = []

    if text_mode:

        text = simpledialog.askstring(
            "Add Text",
            "Type something:"
        )

        if text:

            item = canvas.create_text(
                event.x,
                event.y,
                text=text,
                fill=current_color,
                font=("Arial", brush_size * 4)
            )

            undo_stack.append([item])
            redo_stack.clear()

        return

    last_x = event.x
    last_y = event.y

def draw(event):
global last_x
global last_y
global current_stroke

    if text_mode:
        return

    color = canvas["bg"] if eraser_mode else current_color

    if last_x is not None and last_y is not None:

        line = canvas.create_line(
            last_x,
            last_y,
            event.x,
            event.y,
            fill=color,
            width=brush_size,
            smooth=True,
            capstyle=tk.ROUND,
            joinstyle=tk.ROUND,
            splinesteps=100
        )

        current_stroke.append(line)

    last_x = event.x
    last_y = event.y

def stop_draw(event):
global last_x
global last_y
global current_stroke

    last_x = None
    last_y = None

    if current_stroke:
        undo_stack.append(current_stroke)
        redo_stack.clear()

from PIL import ImageGrab

def save_canvas():

    file = filedialog.asksaveasfilename(
        defaultextension=".fernotes",
        filetypes=[("FerNotes files", "*.fernotes")]
    )

    if not file:
        return

    data = {
        "title": title_entry.get(),
        "items": []
    }

    for item in canvas.find_all():

        if canvas.itemcget(item, "state") == "hidden":
            continue
        item_type = canvas.type(item)

        if item_type == "line":

            data["items"].append({
                "type": "line",
                "coords": canvas.coords(item),
                "fill": canvas.itemcget(item, "fill"),
                "width": canvas.itemcget(item, "width")
            })

        elif item_type == "text":

            data["items"].append({
                "type": "text",
                "coords": canvas.coords(item),
                "text": canvas.itemcget(item, "text"),
                "fill": canvas.itemcget(item, "fill"),
                "font": canvas.itemcget(item, "font")
            })

    with open(file, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=4)


def open_canvas():

    file = filedialog.askopenfilename(
        filetypes=[("FerNotes files", "*.fernotes")]
    )

    if not file:
        return

    with open(file, "r", encoding="utf-8") as f:
        data = json.load(f)

    canvas.delete("all")

    title_entry.delete(0, tk.END)
    title_entry.insert(0, data["title"])

    for item in data["items"]:

        if item["type"] == "line":
            canvas.create_line(
                *item["coords"],
                fill=item["fill"],
                width=float(item["width"]),
                smooth=True,
                capstyle=tk.ROUND,
                joinstyle=tk.ROUND
            )

        elif item["type"] == "text":
            canvas.create_text(
                *item["coords"],
                text=item["text"],
                fill=item["fill"],
                font=item["font"]
            )

    draw_lines()

def update_cursor(event):

    canvas.delete("cursor")

    r = brush_size // 2

    canvas.create_oval(
        event.x - r,
        event.y - r,
        event.x + r,
        event.y + r,
        outline="#b8a8d9",
        width=2,
        tags="cursor"
    )

def draw_lines():

    for y in range(40, HEIGHT, 40):

        canvas.create_line(
            0,
            y,
            WIDTH,
            y,
            fill="#e8e1ff"
        )

toolbar = tk.Frame(root, bg="#f4f0ff")
toolbar.pack(fill="x", pady=8)

title_entry = tk.Entry(
root,
font=("Arial", 20, "bold"),
bg="#fffdf8",
fg="#000000",
bd=0,
justify="center"
)

title_entry.insert(0, "Untitled Notes")

title_entry.pack(fill="x", padx=20, pady=10)

top_colors = tk.Frame(toolbar, bg="#f4f0ff")
top_colors.pack()

bottom_colors = tk.Frame(toolbar, bg="#f4f0ff")
bottom_colors.pack()

preset_colors = [
"#2d2d2d",
"#5a4fcf",
"#ff6ba6",
"#64c8ff",
"#00b894",
"#ff9f43",
"#ffffff"
]

preset_colors_1 = [
"#7f8cff",
"#ff9ec7",
"#8be9fd",
"#b8f2e6",
"#ffd6a5",
"#aaaaaa",
"#633501"
]

for color in preset_colors:

    btn = tk.Button(
        top_colors,
        bg=color,
        width=3,
        height=1,
        relief=tk.FLAT,
        cursor="hand2",
        bd=0,
        command=lambda c=color: set_color(c)
    )

    btn.pack(side=tk.LEFT, padx=6, pady=2)

for color in preset_colors_1:

    btn = tk.Button(
        bottom_colors,
        bg=color,
        width=3,
        height=1,
        relief=tk.FLAT,
        cursor="hand2",
        bd=0,
        command=lambda c=color: set_color(c)
    )

    btn.pack(side=tk.LEFT, padx=6, pady=2)

tools_frame = tk.Frame(toolbar, bg="#f4f0ff")
tools_frame.pack(pady=5)

color_btn = tk.Button(
tools_frame,
text="Colors",
bg="#d8c8ff",
fg="#3d2c5c",
relief=tk.FLAT,
font=("Arial", 10, "bold"),
cursor="hand2",
bd=0,
padx=10,
pady=5,
command=choose_color
)

color_btn.pack(side=tk.LEFT, padx=10)

eraser_btn = tk.Button(
tools_frame,
text="Erase",
bg="#ffd6e0",
fg="#3d2c5c",
relief=tk.FLAT,
font=("Arial", 10, "bold"),
cursor="hand2",
bd=0,
padx=10,
pady=5,
command=toggle_eraser
)

eraser_btn.pack(side=tk.LEFT, padx=10)

text_btn = tk.Button(
tools_frame,
text="Text",
bg="#bde0fe",
fg="#3d2c5c",
relief=tk.FLAT,
font=("Arial", 10, "bold"),
cursor="hand2",
bd=0,
padx=10,
pady=5,
command=toggle_text
)

text_btn.pack(side=tk.LEFT, padx=10)

clear_btn = tk.Button(
tools_frame,
text="New Page",
bg="#ffc6c6",
fg="#3d2c5c",
relief=tk.FLAT,
font=("Arial", 10, "bold"),
cursor="hand2",
bd=0,
padx=10,
pady=5,
command=clear_canvas
)

clear_btn.pack(side=tk.LEFT, padx=10)

open_btn = tk.Button(
tools_frame,
text="Open",
bg="#c7d9ff",
fg="#3d2c5c",
relief=tk.FLAT,
font=("Arial", 10, "bold"),
cursor="hand2",
bd=0,
padx=10,
pady=5,
command=open_canvas
)

open_btn.pack(side=tk.LEFT, padx=10)

save_btn = tk.Button(
tools_frame,
text="Save",
bg="#caffbf",
fg="#3d2c5c",
relief=tk.FLAT,
font=("Arial", 10, "bold"),
cursor="hand2",
bd=0,
padx=10,
pady=5,
command=save_canvas
)

save_btn.pack(side=tk.LEFT, padx=10)

brush*slider = tk.Scale(
tools_frame,
from*=1,
to=20,
orient=tk.HORIZONTAL,
bg="#f4f0ff",
fg="#3d2c5c",
highlightthickness=0,
label="Pen Size",
command=update_brush_size
)

brush_slider.set(5)
brush_slider.pack(side=tk.LEFT, padx=20)

canvas = tk.Canvas(
root,
bg=BG_COLOR,
highlightthickness=0,
width=WIDTH,
height=HEIGHT
)

canvas.pack(fill="both", expand=True)

draw_lines()

canvas.bind("<Button-1>", start_draw)
canvas.bind("<B1-Motion>", draw)
canvas.bind("<ButtonRelease-1>", stop_draw)
canvas.bind("<Motion>", update_cursor)

root.bind("<Control-z>", undo)
root.bind("<Control-y>", redo)
root.bind("c", clear_canvas)
root.bind("e", lambda event: toggle_eraser())

root.mainloop()
