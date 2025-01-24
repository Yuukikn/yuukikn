GUI Calculator

Project Description: Make a simple GUI Calculator for windows with night mode button
![Screenshot 2024-08-11 224301](https://github.com/user-attachments/assets/748f2956-b215-453e-978a-9ba818a55f06)
![Screenshot 2024-08-11 224509](https://github.com/user-attachments/assets/bfa5698e-5973-4f5c-9562-746d8883ff2c)

from tkinter import *
from tkinter import ttk
from PIL import Image
import sys
import re
import keyboard

tray_icon = Image.open(r"C:\Users\albie\PycharmProjects\Calculator_App\.venv\Scripts\calculator.ico")

root = Tk()
root.title("Calculator")
root.geometry("400x500+200+200")
root.rowconfigure(0, weight=0)
root_colour = ""
mainframe = ttk.Frame(root)

menu = Menu(root)
menu.configure(bg="#373737")

night_mode_on = True
toggle_mode_colour = "#373737"
toggle_mode_text_colour = "white"
highlight_colour = "white"
menu.configure(bg="#373737")
width = 16
display = None
user_input = ""
keyboard_pressed = False

#the buttons values are assigned to this dictanary
buttons = {"C": "", "()": "", "%": "", "÷": "", "7": "", "8": "", "9": "", "x": "", "4": "", "5": "", "6": "", "-": "", "1": "", "2": "", "3": "", "+": "", "+/-": "", "0": "", ".": "", "=": ""}

if getattr(sys, 'frozen', False):
    wd = sys._MEIPASS
else:
    wd = ''

current_text = ""

def toggle_bool(night_mode_on_toggle):
    return not night_mode_on_toggle

# Creates a entry wedge to desplay inputs
def display_wedge(root, width):
    global display
    if display == None:
       display = Entry(root, font=("Arial", 46), bg=toggle_mode_colour, fg=toggle_mode_text_colour, highlightcolor=highlight_colour, highlightthickness=1, width=width)
       display.grid(row=1, column=0, columnspan=4, padx=10, pady=10, sticky=N+E+S+W)
    return display

display = display_wedge(root, width)

def on_click(event):
    x, y = event.x, event.y
    root.focus_force()
root.bind("<Button-1>", on_click)

def find_operator_and_number():
    text = display.get()
    split_text = re.findall(r"\+|\-|\÷|\.|\d+|[a-zA-Z]+|\(|\)|\%|\/|\*", text)
    operators = []
    numbers = []
    precentage = []

    for index, character in enumerate(split_text):
        if character == "%":
            move_index_left = 1
            move_index_right = 1

            left_index = split_text[index - move_index_left - 1]
            right_index = split_text[index + move_index_right] if index + 1 < len(split_text) else index
            right_index_plus_2 = split_text[index + move_index_right + 2] if index + 3 < len(
                split_text) else right_index
            precent_number_index = split_text[index - 1]

            right_index_str = str(right_index)

            precentage.append(precent_number_index)

            while left_index.isdigit() or left_index == ".":
                move_index_left -= -1
                left_index = split_text[index - move_index_left]
            if left_index.isdigit() is False and left_index not in ".%":
                operators.append(left_index)

            while left_index.isdigit() is False and left_index != "%" or left_index == ".":
                move_index_left -= -1
                left_index = split_text[index - move_index_left]
            if left_index.isdigit() and left_index != ".":
                numbers.append(left_index)

            if index + 1 < len(split_text):

                while len(split_text) < move_index_right and right_index_str.isdigit() or right_index == ".":
                    move_index_right += 1
                    right_index = split_text[index + move_index_right]
                if right_index_str.isdigit() is False and right_index != ".":
                    operators.append(right_index)

                while str(right_index).isdigit() is False or right_index == ".":
                    move_index_right += 1
                    if index + move_index_right + 2 < len(split_text):
                       right_index = split_text[index + move_index_right]
                    else:
                         break
                if str(right_index).isdigit() and right_index_plus_2 != "%":
                    numbers.append(right_index)

    return operators, numbers, precentage


# compare both operatoers priority to get the number to find the % of
def find_priority_operator_number():
    operators, numbers, precentage = find_operator_and_number()
    text = display.get()
    split_text = re.findall(r"\+|\-|\÷|\.|\d+|[a-zA-Z]+|\(|\)|\%|\/|\*", text)

    precent_sign_count = 0

    for index, character in enumerate(split_text):
        if character == "%":
            index_pear1 = precent_sign_count
            index_pear2 = precent_sign_count + 1
            index_minuse_2 = split_text[index - 2]

            if index_pear2 < len(operators):
                if index_minuse_2 == "(":
                    if index_pear1 < len(operators):
                        operators.pop(index_pear1)
                    if index_pear1 < len(numbers):
                        numbers.pop(index_pear1)
                elif operators[index_pear1] in "x÷*/" or operators[index_pear1] == operators[index_pear2] or operators[index_pear2] == ")":
                    if index_pear2 < len(operators):
                        operators.pop(index_pear2)
                    if index_pear2 < len(numbers):
                        numbers.pop(index_pear2)
                elif operators[index_pear1] in "+-" and operators[index_pear2] in "+-":
                    if index_pear2 < len(operators):
                        operators.pop(index_pear2)
                    if index_pear2 < len(numbers):
                        numbers.pop(index_pear2)
                else:
                    if index_pear1 < len(operators):
                        operators.pop(index_pear1)
                    if index_pear1 < len(numbers):
                        numbers.pop(index_pear1)

                precent_sign_count += 1
    return operators, numbers, precentage

# find the % of the numbers and insert and enclose them back in to text
def find_precentage(text):
    operators, numbers, precentage = find_priority_operator_number()
    text = display.get()
    split_text = re.findall(r"\+|\-|\÷|\.|\d+|[a-zA-Z]+|\(|\)|\%|\/|\*", text)
    for index, character in enumerate(split_text):
        left_index = split_text[index - 2]if index - 2 < len(split_text) else None
        right_index = split_text[index + 1] if index + 1 < len(split_text) else None

        if character == "%":
            left_index_3 = split_text[index - 3] if index - 3 < len(split_text) else None
            left_index_4 = split_text[index - 4] if index - 4 < len(split_text) else left_index_3
            left_index_5 = split_text[index - 5] if index - 5 < len(split_text) else left_index_4
            right_index_3 = split_text[index + 3] if index + 3 < len(split_text) else None

            if len(operators) > 0 and left_index_3 != "%" and left_index_4 != "%" and not left_index in "(/÷" and left_index_4 != "." and not operators[0] in "*x/÷":
                if len(numbers) > 0 and len(precentage) > 0:
                    text = text.replace(f"{precentage[0]}%", f"({precentage[0]}÷100x{numbers[0]})")
                if len(precentage) > 0:
                    precentage.pop(0)
                if len(numbers) > 0:
                    numbers.pop(0)
                operators.pop(0)

            elif  len(operators) > 0 and left_index_4 == "." or len(operators) > 0 and left_index in "+-":
                if len(numbers) > 0 and len(precentage) > 0:
                    text = text.replace(f"{precentage[0]}%", f"(({precentage[0]}÷100x{left_index_5}))")
                if len(precentage) > 0:
                    precentage.pop(0)
                if len(numbers) > 0:
                    numbers.pop(0)
                operators.pop(0)

            elif len(operators) > 0 and right_index == operators[0] and right_index_3 != "%" and not  right_index in "÷/*x":
                print(True)
                if len(numbers) > 0:
                   text = text.replace(f"{precentage[0]}%", f"({precentage[0]}÷100x{numbers[0]})")
                else:
                    text = text.replace(f"{precentage[0]}%", f"({precentage[0]}÷100)")
                if len(precentage) > 0:
                    precentage.pop(0)
                if len(numbers) > 0:
                    numbers.pop(0)
                operators.pop(0)

            else:
                text = text.replace(f"{precentage[0]}%", f"({precentage[0]}÷100)")
                if len(precentage) > 0:
                    precentage.pop(0)
                    if len(operators) > 0:
                        operators.pop(0)
                    if len(numbers) != 0:
                        numbers.pop(0)
    return text

def calculate(text):
    text = find_precentage(text)
    sum = text.replace("x", "*").replace("÷", "/")
    display.delete(0, "end")
    display.insert("end", eval(sum))

# get keyboard inputse
def on_key_pressed(event):
    global user_input, keyboard_pressed
    user_input = event.char
    keyboard_pressed = not keyboard_pressed
    button_click("key")

def check_shift_plus():
    while True:
        if keyboard.is_pressed('shift') and keyboard.is_pressed('+') or keyboard.is_pressed('shift') and keyboard.is_pressed('*'):
            return True
        elif keyboard.is_pressed('shift') and keyboard.is_pressed('%'):
            return  True
        else:
            return False

def button_click(key):
    global keyboard_pressed
    if key in buttons or user_input in buttons or user_input in "*/()" or keyboard.is_pressed("enter") or keyboard.is_pressed("backspace"):
        text = display.get()
        split_text = re.findall(r"\+|\-|\÷|\.|\d+|[a-zA-Z]+|\(|\)|\%|\/|\*", text)
        split_text_revesed = split_text[::-1]
        stack = []

        if keyboard_pressed is True:
            keyboard_pressed = not keyboard_pressed
            key = user_input

        if keyboard.is_pressed("backspace"):
            display.delete(len(display.get()) - 1, END)

        if key.isdigit():
            if len(text) == 0:
               display.insert("end", key)
            elif text[-1] != "%":
                display.insert("end", key)

        elif key == "C":
            display.delete(0, "end")

        elif key == "=" and len(split_text) > 2 or keyboard.is_pressed("enter") and len(split_text) > 2:
            if "(" in text:
                for index, character in enumerate(text):
                    if character == "(":
                        stack.append(character)
                    elif character == ")":
                        stack.pop(0)
                    if index == len(text) - 1:
                         if "(" in stack and text[-1].isdigit() or "(" in stack and text[-1] == "%":
                            display.insert("end", ")")
                         elif len(stack) == 0 and text[-1] in "÷x+.-*/":
                            display.insert("end", "(")
                         elif len(stack) == 0 and text[-1].isdigit() and split_text[-3] != ")":
                           print(True)
                           display.insert("end", "x(")
            calculate(text)

        elif text != ""  and (key in "÷x+-/*" and text[-1].isdigit()) or text != ""  and key in "÷x+-/*" and text[-1] in ")%" :
             display.insert("end", key)

        elif text != "" and key == "%" and text[-1].isdigit():
             display.insert("end", key)

        elif key == "." and len(split_text) == 1 or len(split_text) >= 2 and key == "." and split_text[-2] in "-+x÷()*/":
            if text[-1] not in "()":
               display.insert("end", key)

        # remove or replace a input if the same one is entered twice or another symble is pressed
        elif text != "" and key in "+%*"  and text[-1] in "+%*-x÷./" and check_shift_plus() == True or text != "" and key in "-x÷/."  and text[-1] in "-x÷./+%*":
            if key in "-+x÷.*/" and split_text_revesed[0] != key and split_text_revesed[0] != "%":

                if  key in "+*" and split_text_revesed[0] != key and split_text_revesed[0] != "%" and check_shift_plus():
                    split_text_revesed.remove(split_text_revesed[0])
                    split_text_revesed.insert(0, key)

                elif key in "-/x.÷" and not keyboard.is_pressed("shift"):
                    split_text_revesed.remove(split_text_revesed[0])
                    split_text_revesed.insert(0, key)

            elif key in "-/x.÷" and not keyboard.is_pressed("shift"):
                split_text_revesed.remove(split_text_revesed[0])

            elif key in "-+x÷.*/" and split_text_revesed[0] != "%" and check_shift_plus() or key in "-+x./" and split_text_revesed[0] != "%" and check_shift_plus():
               split_text_revesed.remove(split_text_revesed[0])

            elif key == "%" and split_text_revesed[0] == "%":
               split_text_revesed.remove(split_text_revesed[0])

            elif text[-1] != key and key != "." and key != "%":
                split_text_revesed.insert(0, key)

            elif len(split_text) <= 2 and text[-1] not in "-+x%÷*/" and text[-2].isdigit() and key != "%":
                split_text_revesed.insert(0, key)

            elif len(split_text) >= 3 and text[-1] != "." and (key == "." and split_text[-2].isdigit() is False or key == "." and split_text[-3] in "-+x%÷"):
                 split_text_revesed.insert(0, key)


            split_text_revesed.reverse()
            join_text_list = "".join(split_text_revesed)
            display.delete(0, "end")
            display.insert("end", join_text_list)

        #changes a number to/from positive and negative
        elif text != "" and key == "+/-":
            if len(split_text) == 1 or len(split_text) >= 3 and split_text[-1].isdigit() and split_text[-3].isdigit() and split_text[-2] != ".":
                split_text_revesed.reverse()
                split_text_revesed.insert(-1, "-")
                join_split_text = "".join(split_text_revesed)
                display.delete(0, "end")
                display.insert("end", join_split_text)

            elif len(split_text) >= 3 and  split_text[-2] == "-" and split_text[-3] in "%÷x+.-*/" or len(text) >= 2  and split_text[-2] == "-" and split_text[-3] != ")":
                print(True)
                split_text_revesed.remove("-")
                split_text_revesed.reverse()
                join_split_text = "".join(split_text_revesed)
                display.delete(0, "end")
                display.insert("end", join_split_text)

            elif len(split_text) > 3 and (split_text[-4] == "-"):
                 split_text_revesed.remove("-")
                 split_text_revesed.reverse()
                 join_split_text = "".join(split_text_revesed)
                 display.delete(0, "end")
                 display.insert("end", join_split_text)

            elif len(split_text) == 1 or len(split_text) >= 3 and split_text[-1].isdigit() and split_text[-3].isdigit() and split_text[-2] == ".":
                split_text_revesed.reverse()
                split_text_revesed.insert(-3, "-")
                join_split_text = "".join(split_text_revesed)
                display.delete(0, "end")
                display.insert("end", join_split_text)

            elif len(split_text) > 3 and split_text[-1].isdigit() and split_text[-2] in "()" or len(split_text) > 3 and split_text[-3] == ")":
                split_text_revesed.reverse()
                split_text_revesed.insert(-1, "-")
                join_split_text = "".join(split_text_revesed)
                display.delete(0, "end")
                display.insert("end", join_split_text)

        elif key == "()" or keyboard.is_pressed("(")  or keyboard.is_pressed(")"):
            # iterates over text and puts in and removes ( from the list when a () are found and at the end of the text depending if anything remains in the list () is chosen and inserted in the desplay
            for index, character in enumerate(text):
                if character == "(":
                    stack.append(character)
                elif character == ")":
                    stack.pop(0)
                if index == len(text) - 1:
                    if "(" in stack and text[-1].isdigit() or "(" in stack and text[-1] == "%":
                        display.insert("end", ")")
                    elif len(stack) == 0 and text[-1] in "÷x+.-*/":
                        display.insert("end", "(")
                    elif len(stack) == 0 and text[-1].isdigit():
                        display.insert("end", "x(")

            if len(text) == 0:
                display.insert("end", "(")

            # to stop the same input being entered repedadly eg (((( ))))
            if len(text) >= 1 and text[-1] == "(":
                split_text_revesed.remove("(")
                split_text_revesed.reverse()
                join_split_text = "".join(split_text_revesed)
                display.delete(0, "end")
                display.insert("end", join_split_text)

            elif len(text) >= 1 and text[-1] == ")":
                split_text_revesed.remove(")")
                split_text_revesed.reverse()
                join_split_text = "".join(split_text_revesed)
                display.delete(0, "end")
                display.insert("end", join_split_text)

        display.icursor('end')  # Move the insertion cursor to the end of the text
        display.xview('end')  # Scroll to the end of the text

        text
    keyboard_pressed = not True

# create the buttons
def create_buttons(root, width):
    column = 0
    row = 0
    pading_x = 1
    pading_y = 1
    for key, number in buttons.items():
        if column == 3:
            button = Button(root, font=("Arial", 16), text=str(key), command= lambda key=key: button_click(key), width=width)
            button.grid(row=row + 2, column=column, padx=pading_x, pady=pading_y, sticky="nesw")
            root.columnconfigure(column, weight=1)
            root.rowconfigure(row + 2, weight=1)
            button.configure(bg=toggle_mode_colour, fg=toggle_mode_text_colour)
            column = 0
            row += 1

        else:
            button = Button(root, font=("Arial", 16), text=str(key), command= lambda key=key:  button_click(key), width=width)
            button.grid(row=row + 2, column=column, padx=pading_x, pady=pading_y,sticky="nesw")
            root.columnconfigure(column, weight=1)
            root.rowconfigure(row + 2, weight=1)
            button.configure(bg=toggle_mode_colour, fg=toggle_mode_text_colour)
            column += 1
        buttons[key] = button
    return buttons
create_buttons(root, width)

# create the night mode button
def create_night_mode_buton(root):
    mode_image = "☀"
    night_mode = Button(root, font=("Arial", 16), text=mode_image, command=lambda: night_mode_toggle(root), width=2)
    night_mode.grid(row=0,column=0, padx=1, pady=0, sticky="nw")
    night_mode.configure(bg=toggle_mode_colour, fg=toggle_mode_text_colour)
    return night_mode

#toggles the colour of the bacground and numbers when the night mode putton is pressed
def night_mode_toggle(root):
    global toggle_mode_colour, toggle_mode_text_colour, highlight_colour, night_mode_on, root_colour
    night_mode_on = not night_mode_on
    night_mode_button = create_night_mode_buton(root)
    display = display_wedge(root, width)
    if night_mode_on == True:
       root_colour = "white"
       night_mode_button.configure(bg="white", fg="black", text="☾")
       display.configure(bg="white", fg="black", highlightcolor="black")
       for key, button in buttons.items():
           buttons[key].configure(bg="white", fg="black")

    else:
       root_colour = "#373737"
       night_mode_button.configure(bg="#373737", fg="white", text="☀" )
       display.configure(bg="#373737", fg="white", highlightcolor="white")
       for key, button in buttons.items():
           buttons[key].configure(bg="#373737", fg="white")
    root.configure(bg=root_colour)
    return toggle_mode_colour, toggle_mode_text_colour, highlight_colour, root_colour

root.bind("<Key>", on_key_pressed)
night_mode_toggle(root)
root.mainloop()



