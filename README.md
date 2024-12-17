import tkinter as tk
from tkinter import ttk
import math
from datetime import datetime
import tkinter.filedialog as filedialog
import tkinter.messagebox as messagebox
import matplotlib.pyplot as plt
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg


# Function to calculate qu_drained
def calculate_qu_drained():
    # Get user inputs from Entry widgets and convert them to float
    C_drained = float(entry_vars["C_drained"].get())
    Φ_drained = float(entry_vars["Φ_drained"].get())
    γ_bulk = float(entry_vars["γ_bulk"].get())
    C_undrained = float(entry_vars["C_undrained"].get())
    Φ_undrained = float(entry_vars["Φ_undrained"].get())
    B = float(entry_vars["Width B"].get())
    L = float(entry_vars["Length L"].get())
    Df = float(entry_vars["Depth Df"].get())
    β = float(entry_vars["Inclination β"].get())
    γw = float(entry_vars["Water γw"].get())
    wt = float(entry_vars["Water table wt"].get())
    SF = float(entry_vars["Safety factor"].get())
    Column_load = float(entry_vars["Column load"].get())
    Eccentricity_ex = float(entry_vars["Eccentricity ex"].get())
    Eccentricity_ey = float(entry_vars["Eccentricity ey"].get())
    
    
    Bp = B-2*Eccentricity_ex
    Lp = L-2*Eccentricity_ey
    qmax_B = (Column_load/(B*L))*(1+((6*Eccentricity_ex)/B))
    qmin_B = (Column_load/(B*L))*(1-((6*Eccentricity_ex)/B))
    qref_B = ((3/4)*qmax_B) + ((1/4)*qmin_B)
    
    qmax_L = (Column_load/(B*L))*(1+((6*Eccentricity_ex)/L))
    qmin_L = (Column_load/(B*L))*(1-((6*Eccentricity_ex)/L))
    qref_L = ((3/4)*qmax_L) + ((1/4)*qmin_L) 
    
    # Calculate bearing capacity factor Nq for drained consitions
    angle_degrees = 45 + Φ_drained / 2
    tan_term = math.tan(math.radians(angle_degrees))**2
    exponent_term = math.exp(math.pi * math.tan(math.radians(Φ_drained)))
    Nq = tan_term * exponent_term
    Nc = (Nq - 1) / math.tan(math.radians(Φ_drained))
    Nγ = 2 * (Nq + 1) * math.tan(math.radians(Φ_drained))
    
    # Calculate bearing capacity factor Nq for undrained conditions
    angle_degrees_undrained = 45 + Φ_undrained / 2
    tan_term_undrained = math.tan(math.radians(angle_degrees_undrained)) ** 2
    exponent_term_undrained = math.exp(math.pi * math.tan(math.radians(Φ_undrained)))
    Nq_undrained = tan_term_undrained * exponent_term_undrained
    
    # Calculate Nc and Nγ for undrained conditions
    # Calculate Nc and Nγ for undrained conditions
    Nc_undrained = 5.14
    Nγ_undrained = 0


    # Calculate the shape factors
    Fcs = 1+(Bp/Lp)*(Nq/Nc)
    Fqs = 1+(Bp/Lp)*math.tan(math.radians(Φ_drained))
    Fγs = 1-0.4*(Bp/Lp)
    
    # Calculate the shape factors for undrained conditions
    Fcs_undrained = 1 + (Bp/Lp) * (Nq_undrained / Nc_undrained)
    Fqs_undrained = 1 + (Bp/Lp) * math.tan(math.radians(Φ_undrained))
    Fγs_undrained = 1 - 0.4 * (Bp/Lp)

    # Calculate the depth factors
    #if Φ_undrained == 0:
    Fcd_undrained = 1+0.4*(Df/B)
    Fqd_undrained = 1
    Fγd_undrained = 1

    Fqd_drained = 1+((Df/B)*(2*(math.tan(math.radians(Φ_drained)))*((1-math.sin(math.radians(Φ_drained))))**2))
    Fcd_drained = Fqd_drained
    Fγd_drained = 1
    
    #Inclination
    Fci = (1-β/90)**2
    Fqi=Fci
    Fγi =(1-β/Φ_drained) **2  
    q = γ_bulk*Df
    
    # Inclination factors for undrained conditions
    Fci_undrained = (1 - β / 90) ** 2
    Fqi_undrained = Fci_undrained
        # Inclination factors for undrained conditions
    if Φ_undrained != 0:
        Fγi_undrained = (1 - β / Φ_undrained) ** 2
    else:
        # Handle the case when Φ_undrained is zero
        Fγi_undrained = 0  # or any other appropriate value

    γ_used_drained = 0
    if wt ==0:
        γ_used_drained = γ_bulk-γw
        γ_used_undrained = γ_bulk
        q_drained = γ_bulk * Df
        q_undrained = γ_bulk * Df
    elif 0<wt<=Df: #wt=0.5
        γ_used_drained = γ_bulk-γw
        γ_used_undrained = γ_bulk
        q_drained = (γ_bulk * wt) + ((γ_bulk-γw) * (Df-wt))
        q_undrained = γ_bulk * Df
    elif Df<wt<=(Df+B):
        d=wt-Df
        γ_used_drained = ((γ_bulk)*(d) + (γ_bulk-γw)*(B-d)) /B
        γ_used_undrained = γ_bulk
        q_drained = (γ_bulk-γw) * Df
        q_undrained = γ_bulk * Df
    elif wt>Df+B:        
        γ_used_drained = γ_bulk
        γ_used_undrained = γ_bulk
        q_drained = (γ_bulk) * Df
        q_undrained = γ_bulk * Df
    else:
        print('MAZA HONALEKAAA, may manna ma2khoze bi taree2a sa7i7a, i3adat al nazar!')
        
    q_undrained = γ_bulk * Df
    #print(Df)
    #print(wt)
    #print(B)
    # Calculate qu_drained
    qu_drained = C_drained * Nc * Fcs * Fcd_drained + q_drained * Nq * Fqs * Fqd_drained * Fqi + (0.5) * γ_used_drained * Bp * Nγ * Fγs * Fγd_drained * Fγi
    #print(C_drained * Nc * Fcs * Fcd_drained)
    #print (q_drained)
    #print(q_drained * Nq * Fqs * Fqd_drained * Fqi )
    #print(0.5*γ_used_drained * Bp * Nγ * Fγs * Fγd_drained * Fγi)
    qall_drained = qu_drained/SF
    Qall_drained = math.cos(math.radians(β))*(qall_drained*(Bp*Lp))
    # Calculate qu_undrained
    qu_undrained = C_undrained * Nc_undrained * Fcs_undrained  * Fcd_undrained  + q_undrained * Nq_undrained * Fqs_undrained * Fqd_undrained * Fqi + (0.5) * γ_bulk * Bp * Nγ_undrained * Fγs_undrained * Fγd_undrained * Fγi
    qall_undrained = qu_undrained/SF
    Qall_undrained = math.cos(math.radians(β))*(qall_undrained*(Bp*Lp))
    #Skempton approach in clays D/B<= 2.5
    # Skempton approach in clays D/B<= 2.5, skempton is applicable if we have low friction angles
    if Df/B <= 2.5 and (Φ_undrained<=2 or Φ_drained<=2) :
        qsk_drained = (1 + 0.2 * (Df/B)) * (1 + 0.2 * (Bp/Lp)) * 5.14 * C_drained + q_drained
        qsk_undrained = (1 + 0.2 * (Df/B)) * (1 + 0.2 * (Bp/Lp)) * 5.14 * C_undrained + q_undrained
        
        # Clear previous messages in result_text3
        result_text3.config(state=tk.NORMAL)
        result_text3.delete(1.0, tk.END)
        result_text3.config(state=tk.DISABLED)
        
        # Update the result label with the calculated value including both qu_undrained and qall
        result_text3.config(state=tk.NORMAL)
        result_text3.insert(tk.END, f"Ultimate drained qu' = {qsk_drained:.3f} kPa\nUltimate Undrained qu' = {qsk_undrained:.3f} kPa")
        result_text3.config(state=tk.DISABLED)
    else:
        # Clear previous messages in result_text3
        result_text3.config(state=tk.NORMAL)
        result_text3.delete(1.0, tk.END)
        result_text3.config(state=tk.DISABLED)
        
        # Configure the tag for green color
        result_text3.tag_configure('green', foreground='green')
        
        # Update the result label with the message indicating Skempton approach is not applicable
        result_text3.config(state=tk.NORMAL)
        result_text3.insert(tk.END, 'Skempton is not applicable, check manual\n', 'green')
        result_text3.config(state=tk.DISABLED)
    # Clear previous messages in result_text3
    result_text4.config(state=tk.NORMAL)
    result_text4.delete(1.0, tk.END)
    result_text4.config(state=tk.DISABLED)    
    # Check with external loading
    if qref_B > qall_drained or qref_L> qall_drained or Column_load>Qall_drained:
        result_text4.config(state=tk.NORMAL)
        result_text4.insert(tk.END, 'Footing is unsafe in drained condition\n', 'red')
        result_text4.config(state=tk.DISABLED)
        if Eccentricity_ex>=B/6 or Eccentricity_ey>=L/6:
            result_text4.config(state=tk.NORMAL)
            result_text4.insert(tk.END, 'Footing is overturning (drained)\n', 'darkviolet')
            result_text4.config(state=tk.DISABLED)
    else:
        result_text4.config(state=tk.NORMAL)
        result_text4.insert(tk.END, 'Footing is safe in drained condition\n')
        result_text4.config(state=tk.DISABLED)
    
    if qref_B > qall_undrained or qref_L> qall_undrained or Column_load>Qall_undrained:
        result_text4.config(state=tk.NORMAL)
        result_text4.insert(tk.END, 'Footing is unsafe in undrained condition\n', 'red')
        result_text4.config(state=tk.DISABLED)
        if Eccentricity_ex>=B/6 or Eccentricity_ey>=L/6:
            result_text4.config(state=tk.NORMAL)
            result_text4.insert(tk.END, 'Footing is overturning (undrained)\n', 'darkviolet')
            result_text4.config(state=tk.DISABLED)
    else:
        result_text4.config(state=tk.NORMAL)
        result_text4.insert(tk.END, 'Footing is safe in undrained condition\n')
        result_text4.config(state=tk.DISABLED)
    
    # Define the tag for red color
    result_text4.tag_configure('red', foreground='red')
    # Define the tag for red color
    result_text4.tag_configure('darkviolet', foreground='darkviolet')
    # Update the result label with the calculated value including both qu_drained and qall
    result_text1.config(state=tk.NORMAL)
    result_text1.delete(1.0, tk.END)
    result_text1.insert(tk.END, f"Ultimate drained B.C. qu' = {qu_drained:.3f} kPa\nAllowable drained B.C. qall = {qall_drained:.3f} kPa\nAllowable drained force Qall = {Qall_drained:.3f} kN")
    result_text1.config(state=tk.DISABLED)

    # Update the result label with the calculated value including both qu_undrained and qall
    result_text2.config(state=tk.NORMAL)
    result_text2.delete(1.0, tk.END)
    result_text2.insert(tk.END, f"Ultimate undrained B.C. qu' = {qu_undrained:.3f} kPa\nAllowable undrained B.C. qall = {qall_undrained:.3f} kPa\nAllowable undrained force Qall = {Qall_undrained:.3f} kN")
    result_text2.config(state=tk.DISABLED)

    # Clear previous drawing
    canvas.delete("all")
    
    # Draw the footing
    x_center = 600  # Center of the canvas
    y_bottom = 300  # Bottom of the canvas
    width = float(entry_vars["Width B"].get()) * 100  # Adjust the factor to scale the width
    depth = 0.5*100
    thickness =50# width / 4  # Thickness is B/4
    
    # Draw horizontal rectangle
    canvas.create_rectangle(x_center - width/2, y_bottom - depth - thickness, x_center + width/2, y_bottom - depth, outline="black", fill="lightgrey")
    
    # Calculate dimensions of the vertical rectangle (mimicking a column)
    column_width = width / 5  # Adjust the factor to control the width of the column relative to the bottom rectangle
    column_height = depth * 5  # Adjust the factor to control the height of the column relative to the bottom rectangle
    # Draw vertical rectangle (mimicking a column)
    canvas.create_rectangle(x_center - column_width/2, y_bottom - depth - (column_height/1.5), x_center + column_width/2, y_bottom - depth, outline="black", fill="red", width=2)  
    # Draw straight line above the rectangle to mimic a force
    force_line = canvas.create_line(x_center, y_bottom - depth - (column_height/1.5), x_center, y_bottom - depth - (column_height/1.5) + 50, fill="blue", width=2)
    # Draw arrowhead at the bottom of the line
    arrowhead = canvas.create_polygon(x_center - 5, y_bottom - depth - (column_height/1.5) + 45, x_center + 5, y_bottom - depth - (column_height/1.5) + 45, x_center, y_bottom - depth - (column_height/1.5) + 55, fill="blue")
    # Draw horizontal line at the bottom of the footing
    canvas.create_line(0, y_bottom-depth, x_center*2, y_bottom-depth, fill="black", width=2)
    canvas.create_line(0, y_bottom-depth-50*Df, x_center*2, y_bottom-depth-50*Df, fill="red", width=2)
    canvas.create_line(0, y_bottom-depth-50*Df+50*wt, x_center*2, y_bottom-depth-50*Df+50*wt, fill="blue", width=2, dash=(4, 4))

    
    canvas.create_line(x_center-width/2, y_bottom-depth, x_center ,y_bottom-depth+width/2, fill="green", width=2)
    canvas.create_line(x_center, y_bottom-depth +width/2, x_center+width/2 ,y_bottom-depth, fill="green", width=2)
    
    canvas.create_line(x_center-width/2, y_bottom-depth, x_center-width ,1.2*(y_bottom-depth+width/2), fill="orange", width=2)
    canvas.create_line(x_center-width, 1.2*(y_bottom-depth+width/2), 0 , y_bottom-depth , fill="orange", width=2)

    canvas.create_line(x_center-width/2 +width, y_bottom-depth, x_center+width ,1.2*(y_bottom-depth+width/2), fill="orange", width=2)
    canvas.create_line(x_center+width, 1.2*(y_bottom-depth+width/2), 2*x_center , y_bottom-depth , fill="orange", width=2)
    canvas.create_arc(1200, y_bottom-depth +width, x_center, 250, start=180, extent=180, outline="magenta", width=2, style="arc")
    canvas.create_arc(0, y_bottom-depth +width, x_center, 250, start=180, extent=180, outline="magenta", width=2, style="arc")

        # Add legends to the canvas
    canvas.create_text(x_center, y_bottom - depth - 20, text="Footing", fill="black", font=("Helvetica", 10, "bold"))
    canvas.create_text(x_center, y_bottom - depth - column_height - 20, text="Column", fill="black", font=("Helvetica", 10, "bold"))
    canvas.create_text(x_center*0.5, y_bottom - depth + width/4, text="             Zone III\n Overburden pressure", fill="orange", font=("Helvetica", 10, "bold"))
    canvas.create_text(x_center*1.5, y_bottom - depth + width/4, text="             Zone III\n Overburden pressure", fill="orange", font=("Helvetica", 10, "bold"))
    canvas.create_text(x_center, y_bottom - depth - 50*Df - 20, text="Df", fill="red", font=("Helvetica", 10, "bold"))
    canvas.create_text(10, y_bottom - depth - 50*Df + 50*wt - 2, text="∇", fill="blue", font=("Helvetica", 18, "bold"))
    canvas.create_text(x_center - width/2, y_bottom - depth + width/2 + 20, text="   Zone II\nSlip circle", fill="magenta", font=("Helvetica", 10, "bold"))
    canvas.create_text(x_center + width/2, y_bottom - depth + width/2 + 20, text="   Zone II\nSlip circle", fill="magenta", font=("Helvetica", 10, "bold"))
    canvas.create_text(x_center,y_bottom  , text="      Zone I\ncompression", fill="green", font=("Helvetica", 10, "bold"))
# Function to set default values
def set_default_values():
    default_values = {
        "C_drained": 10, "Φ_drained": 30, "γ_bulk": 18,
        "C_undrained": 30, "Φ_undrained": 0.0, "Width B": 2,
        "Length L": 3, "Depth Df":1, "Inclination β":0,
        "Water table wt":0.0, "Water γw":10, "Safety factor":3,
        "Column load":800,"Eccentricity ex":0.0,"Eccentricity ey":0.0
    }    
    # Set default values in Entry widgets
    for var, value in default_values.items():
        entry_vars[var].delete(0, tk.END)
        entry_vars[var].insert(0, str(value))
        
# Function to set default values
def set_default_values1():
    default_values = {
        "C_drained": 20, "Φ_drained": 1, "γ_bulk": 18,
        "C_undrained": 30, "Φ_undrained": 0.0, "Width B": 2,
        "Length L": 3, "Depth Df":1, "Inclination β":0,
        "Water table wt":0.0, "Water γw":10, "Safety factor":3,
        "Column load":800,"Eccentricity ex":0.0,"Eccentricity ey":0.0
    }    
    # Set default values in Entry widgets
    for var, value in default_values.items():
        entry_vars[var].delete(0, tk.END)
        entry_vars[var].insert(0, str(value))        


# Function to set default values
def set_default_values2():
    default_values = {
        "C_drained": 1, "Φ_drained": 30, "γ_bulk": 18,
        "C_undrained": 1, "Φ_undrained": 30, "Width B": 2,
        "Length L": 3, "Depth Df":1, "Inclination β":0,
        "Water table wt":0.0, "Water γw":10, "Safety factor":3,
        "Column load":800,"Eccentricity ex":0.0,"Eccentricity ey":0.0
    }    
    # Set default values in Entry widgets
    for var, value in default_values.items():
        entry_vars[var].delete(0, tk.END)
        entry_vars[var].insert(0, str(value))
# Create the main window
root = tk.Tk()
root.title("Shallow foundation bearing capacity")
root.geometry("1900x1900")
root.configure(bg="snow")
# Load the image
image = tk.PhotoImage(file="MEY.png")  # Replace "image_file.png" with the actual filename of your image
image = image.subsample(2, 2)

image1 = tk.PhotoImage(file="BC_soils.png")  # Replace "image_file.png" with the actual filename of your image
image1 = image1.subsample(1, 1)

# Create a label for the section title
title_label = tk.Label(root, text="Geometric properties", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=50, y=20)

# Create a label for the section title
title_label = tk.Label(root, text="Hydrology", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=50, y=170)

# Create a label for the section title
title_label = tk.Label(root, text="Safety factor", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=50, y=260)

# Create a label for the section title
title_label = tk.Label(root, text="Column load", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=50, y=320)

# Create a label for the section title
title_label = tk.Label(root, text="Biaxial eccentricities", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=300, y=260)

# Create a label for the section title
title_label = tk.Label(root, text="Soil drained conditions", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=300, y=20)

# Create a label for the section title
title_label = tk.Label(root, text="Soil undrained conditions", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=300, y=170)

# Create a label for the section title
title_label = tk.Label(root, text="Drained analysis", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=650, y=20)

# Create a label for the section title
title_label = tk.Label(root, text="Undrained analysis", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=650, y=170)

# Create a label for the section title
title_label = tk.Label(root, text="Skempton analysis", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=1020, y=20)

# Create a label for the section title
title_label = tk.Label(root, text="Foundation safety check", font=("Arial", 14, "bold italic underline"), fg="#00008B")
title_label.place(x=990, y=170)


# Create labels and entry widgets for each variable
variables = [
    ("Width B", 50, 50), ("Length L", 50, 80), ("Depth Df", 50, 110),
    ("Inclination β", 50, 140), ("Water γw", 50, 200), ("Water table wt", 50, 230),
    ("Safety factor", 50, 290),("Column load", 50, 350), ("C_drained", 300, 50), ("Φ_drained", 300, 80),
    ("γ_bulk", 300, 110), ("C_undrained", 300, 200), ("Φ_undrained", 300, 230),("Eccentricity ex", 300, 290),("Eccentricity ey", 300, 320)
]
entry_vars = {}

# Adjust the width of the entry widgets
entry_width = 8  # Adjust this value to your preference

for var, x, y in variables:
    label = tk.Label(root, text=f"{var}:")
    label.place(x=x, y=y)

    entry = tk.Entry(root, width=entry_width)
    entry.place(x=x + 100, y=y)
    entry_vars[var] = entry

# Create the first text widget for displaying results
result_text1 = tk.Text(root, width=45, height=5, state=tk.DISABLED)
result_text1.place(x=550, y=50)

# Create the second text widget for displaying results
result_text2 = tk.Text(root, width=45, height=5, state=tk.DISABLED)
result_text2.place(x=550, y=200)

# Create the second text widget for displaying results
result_text3 = tk.Text(root, width=40, height=5, state=tk.DISABLED)
result_text3.place(x=935, y=50)

# Create the second text widget for displaying results
result_text4 = tk.Text(root, width=40, height=5, state=tk.DISABLED)
result_text4.place(x=935, y=200)


result_text5 = tk.Text(root, width=20, height=20)
result_text5.place(x=350, y=650)

def save_result_notes():
    # Get the content of the result_text5 widget
    notes_content = result_text5.get("1.0", tk.END)
    # Open a file dialog to choose the file path and name
    file_path = filedialog.asksaveasfilename(defaultextension=".txt", filetypes=[("Text files", "*.txt")])
    # Write the content to the chosen file
    if file_path:
        with open(file_path, "w") as file:
            file.write(notes_content)
        messagebox.showinfo("Saved", "Data saved successfully.")

# Create a button to save notes from result_text5
save_notes_button = tk.Button(root, text="Plot points", command=save_result_notes, bg="lightblue", fg="black", font=("Arial", 12, "bold"), relief=tk.RAISED)
save_notes_button.place(x=382, y=620)


# Create a canvas for drawing
canvas = tk.Canvas(root, width=1200, height=650, bg="white")
canvas.place(x=550, y=320)



# Create a custom style for the button
style = ttk.Style()
style.configure("Sexy.TButton", foreground="white", background="crimson", font=("Arial", 18, "bold"))

# Create the button with the custom style
calculate_button = ttk.Button(root, text="Calculate", style="Sexy.TButton", command=calculate_qu_drained)
calculate_button.place(x=50, y=530)
# Function to update the time clock label
def update_clock():
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    current_date = now.strftime("%Y-%m-%d")
    clock_label.config(text=f"Date: {current_date}   Time: {current_time}")
    root.after(1000, update_clock)  # Schedule the function to run again after 1000ms (1 second)

# Create a label for the time clock
clock_label = tk.Label(root, font=("Arial", 14), bg="snow")
clock_label.place(relx=0.16, rely=0.99, anchor='se')

# Update the time clock label
update_clock()

# Create a label for the image
image_label = tk.Label(root, image=image)
image_label.place(x=1280, y=10)  # Adjust the coordinates as needed


# Create a label for SI units
units_label = tk.Label(root, text="Units: KN, m, degrees", font=("Arial", 14), bg="snow")
units_label.place(relx=0.96, rely=0.99, anchor='se')

# Button to set default values
default_button = tk.Button(root, text="By default", command=set_default_values, bg="lightgray")
default_button.place(x=50, y=590)

# Button to set default values
default_button = tk.Button(root, text="By default clays", command=set_default_values1, bg="lightcoral")
default_button.place(x=50, y=620)

# Button to set default values
default_button = tk.Button(root, text="By default sands", command=set_default_values2, bg="gold")
default_button.place(x=50, y=650)


# Create labels and entry widgets for each variable
for var, x, y in variables:
    label = tk.Label(root, text=f"{var}:")
    label.place(x=x, y=y)

    entry = tk.Entry(root, width=entry_width)
    entry.place(x=x + 100, y=y)
    entry_vars[var] = entry

    # Create buttons for increasing and decreasing the value
    plus_button = tk.Button(root, text="+", command=lambda v=entry_vars[var]: [increment_value(v), calculate_qu_drained()], width=1, height=1, borderwidth=1, relief="solid", bg="lightcyan", fg="black", font=("Helvetica", 8)) 
    plus_button.place(x=x + 170, y=y-1)

    minus_button = tk.Button(root, text="-", command=lambda v=entry_vars[var]: [decrement_value(v), calculate_qu_drained()], width=1, height=1, borderwidth=1, relief="solid", bg="lightcoral", fg="black", font=("Helvetica", 8))
    minus_button.place(x=x + 204, y=y-1)

# Define functions to increment and decrement the value of a variable
def increment_value(entry):
    current_value = float(entry.get())
    entry.delete(0, tk.END)
    entry.insert(0, str(current_value + 0.5))

def decrement_value(entry):
    current_value = float(entry.get())
    if current_value > 0:
        entry.delete(0, tk.END)
        entry.insert(0, str(current_value - 0.5))

    
    
   
    
def open_new_window():
    # Create a new Tk instance for the new window
    new_window = tk.Toplevel(root)  # Use Toplevel to create a new window
    new_window.title("Report generation and analysis")
    # Create a canvas for drawing
    canvas = tk.Canvas(new_window, width=1200, height=1000, bg="white")
    canvas.pack()
    # Create a text box for notes
    notes_text = tk.Text(new_window, height=40, width=70)
    notes_text.pack()
    notes_text.place(x=20, y=20)
    # Create a label for image1
    image_label1 = tk.Label(new_window, image=image1)
    image_label1.pack()
    image_label1.place(x=600, y=0)  # Adjust the coordinates as needed
    # Create a button to save notes in the new window
    generate_report_button = tk.Button(new_window, text="Generate report", command=lambda: save_notes(new_window, notes_text))
    generate_report_button.pack()
    generate_report_button.place(x=250, y=800)
    # Run the main loop for the new window
    new_window.mainloop()

def open_plot_window():
    # Read data from the file
    try:
        with open('ploty.txt', 'r') as file:
            data = file.readlines()
            x_values = []
            y_values = []
            for line in data:
                values = line.strip().split()
                if len(values) >= 2:
                    x, y = map(float, values)
                    x_values.append(x)
                    y_values.append(y)
                else:
                    print("Skipping line:", line)  # or handle the error in some other way
    except FileNotFoundError:
        messagebox.showerror("File Not Found", "The file 'ploty.txt' was not found.")
        return
    # Create a new Tk instance for the plot window
    plot_window = tk.Toplevel(root)
    plot_window.title("Plot")
    # Create a Matplotlib figure
    fig = Figure(figsize=(8, 6), dpi=100)
    ax = fig.add_subplot(111)
    # Plot the data
    ax.plot(x_values, y_values)
    ax.scatter(x_values, y_values, color='red')
    ax.set_ylabel('Bearing strength')
    ax.set_xlabel('Inclination')
    ax.set_title('Parametric study')
    ax.grid(True)
    # Create a canvas to display the plot in the Tkinter window
    canvas = FigureCanvasTkAgg(fig, master=plot_window)
    canvas.draw()
    canvas.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)

# Function to save notes to a file
def save_notes(window, notes_text):
    # Get the content of the notes_text widget
    notes_content = notes_text.get(1.0, tk.END)
    # Open a file dialog to choose the file path and name
    file_path = filedialog.asksaveasfilename(defaultextension=".txt", filetypes=[("Text files", "*.txt")])
    # Write the content to the chosen file
    if file_path:
        with open(file_path, "w") as file:
            file.write(notes_content)
        messagebox.showinfo("Saved", "Notes saved successfully.")
        window.destroy()  # Close the new window after saving notes

# Create a button to open the new window
help_button = tk.Button(root, text="Report writing", command=open_new_window)
help_button.place(x=50, y=920)

plot_button = tk.Button(root, text="Plotting", command=open_plot_window, bg="lightblue", fg="black", font=("Arial", 12, "bold"), relief=tk.RAISED)
plot_button.place(x=50, y=850)


root.mainloop()
    
    
    
    
    
    
    
    
    
'''
def open_plot_window():
    # Read data from the file
    try:
        with open('ploty.txt', 'r') as file:
            data = file.readlines()
            x_values = []
            y_values = []
            for line in data:
                x, y = map(float, line.strip().split())
                x_values.append(x)
                y_values.append(y)
    except FileNotFoundError:
        messagebox.showerror("File Not Found", "The file 'ploty.txt' was not found.")
        return

    # Create a new Tk instance for the plot window
    plot_window = tk.Toplevel(root)
    plot_window.title("Plot")

    # Create a canvas for plotting
    plot_canvas = tk.Canvas(plot_window, width=1000, height=1000, bg="white")
    plot_canvas.pack()

    # Find minimum and maximum values for scaling
    min_x = min(x_values)
    max_x = max(x_values)
    min_y = min(y_values)
    max_y = max(y_values)

    # Calculate scaling factors
    scale_x = 780 / (max_x - min_x)  # Scale to fit within canvas width
    scale_y = 580 / (max_y - min_y)  # Scale to fit within canvas height

    # Plot the grid and ticks
    for i in range(10):
        # Vertical grid lines and ticks
        x = 10 + i * (780 / 10)
        plot_canvas.create_line(x, 10, x, 590, fill="lightgray", dash=(2, 2))
        plot_canvas.create_text(x, 595, text="{:.1f}".format(min_x + i * (max_x - min_x) / 10), anchor="n")

        # Horizontal grid lines and ticks
        y = 590 - i * (580 / 10)
        plot_canvas.create_line(10, y, 790, y, fill="lightgray", dash=(2, 2))
        plot_canvas.create_text(5, y, text="{:.1f}".format(min_y + i * (max_y - min_y) / 10), anchor="e")

    # Plot the data on the canvas
    for i in range(len(x_values) - 1):
        # Scale coordinates to fit within the canvas
        x0 = (x_values[i] - min_x) * scale_x + 10  # Add padding for left margin
        y0 = 590 - (y_values[i] - min_y) * scale_y  # Invert y-axis and add padding for bottom margin
        x1 = (x_values[i + 1] - min_x) * scale_x + 10
        y1 = 590 - (y_values[i + 1] - min_y) * scale_y
        plot_canvas.create_line(x0, y0, x1, y1, fill="blue")

    # Add axis labels
    plot_canvas.create_text(400, 580, text="Second Column", fill="black")
    plot_canvas.create_text(20, 300, text="First Column", fill="black", angle=90)



# Function to save notes to a file
def save_notes(window, notes_text):
    # Get the content of the notes_text widget
    notes_content = notes_text.get(1.0, tk.END)

    # Open a file dialog to choose the file path and name
    file_path = filedialog.asksaveasfilename(defaultextension=".txt", filetypes=[("Text files", "*.txt")])

    # Write the content to the chosen file
    if file_path:
        with open(file_path, "w") as file:
            file.write(notes_content)
        messagebox.showinfo("Saved", "Notes saved successfully.")
        window.destroy()  # Close the new window after saving notes

# Create a button to open the new window
help_button = tk.Button(root, text="Report writing", command=open_new_window)
help_button.place(x=50, y=920)
'''



















# Create a button to save notes
#save_button = tk.Button(root, text="Generate report", command=save_notes)
#save_button.place(x=1625, y=900)
