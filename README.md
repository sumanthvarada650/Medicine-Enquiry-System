# Medicine-Enquiry-System
Built with Python, Tkinter, and Matplotlib, this app simplifies medicine research. Search by name or symptom to view details from a local CSV. Its key feature is dynamic scatter plots that visually compare cost vs. efficacy for related drugs, helping users choose the best value treatment without complex medical knowledge.
# Medicine-Enquiry-System
"""Built with Python, Tkinter, and Matplotlib, this app simplifies medicine research. Search by name or symptom to view details from a local CSV. Its key feature is dynamic scatter plots that visually compare cost vs. efficacy for related drugs, helping users choose the best value treatment without complex medical knowledge.
"""
# The Medicine Enquiry System 

# We use built-in Python libraries: tkinter for the GUI and csv for data handling.
# We also use the 'matplotlib' library for plotting the comparison curves.
import tkinter as tk
from tkinter import ttk, scrolledtext, messagebox
import csv
import matplotlib.pyplot as plt
import numpy as np # Imported for robust calculation of min/max values

# Global variable to store the medicine data loaded from CSV
MEDICINE_DATA = []

# ----------------------------------------------------------------------
# 1. DATA LOADING
# ----------------------------------------------------------------------

def load_data_from_csv(filepath="medicine_data.csv"):
    """
    Loads medicine data from a CSV file into a global list of dictionaries.
    Returns True on success, False on failure.
    """
    global MEDICINE_DATA
    MEDICINE_DATA = []
    
    try:
        with open(filepath, mode='r', newline='', encoding='utf-8') as file:
            # The csv.DictReader reads the first line as fieldnames (keys)
            # and then treats each subsequent row as a dictionary.
            reader = csv.DictReader(file)
            for row in reader:
                # Convert numerical fields from string to float for plotting
                try:
                    row['Efficacy'] = float(row['Efficacy'])
                    row['Cost_per_dose'] = float(row['Cost_per_dose'])
                except ValueError:
                    # Skip rows with invalid numerical data
                    continue
                MEDICINE_DATA.append(row)
        return True
    except FileNotFoundError:
        # Display a clear error message in a pop-up window
        messagebox.showerror("Data Error", 
                             f"Error: The data file '{filepath}' was not found.\n\n"
                             f"Please ensure 'medicine_data.csv' is in the same folder as the script.")
        return False

# ----------------------------------------------------------------------
# 2. CORE LOGIC
# ----------------------------------------------------------------------

def get_medicine_details(medicine_name, output_text_widget):
    """
    Searches the loaded data for a medicine and updates the GUI output.
    """
    # Clear previous output
    output_text_widget.config(state=tk.NORMAL)
    output_text_widget.delete('1.0', tk.END)
    
    search_name = medicine_name.strip().title()
    found_medicine = None
    
    # Simple linear search through the list of medicine dictionaries
    for medicine in MEDICINE_DATA:
        # Check for case-insensitive match on Name
        if medicine['Name'].title() == search_name:
            found_medicine = medicine
            break

    if found_medicine:
        # Display details in the Text widget
        output = "--- Medicine Details ---\n"
        output += f"Name: {found_medicine['Name']}\n"
        output += f"Used For: {found_medicine['Symptom']}\n"
        output += f"Manufacturer: {found_medicine['Manufacturer']}\n"
        output += f"Storage: {found_medicine['Storage']}\n"
        output += f"Efficacy Rating (1-10): {found_medicine['Efficacy']:.1f}\n"
        output += f"Estimated Cost per Dose: ${found_medicine['Cost_per_dose']:.2f}\n"
        output += f"Description: {found_medicine['Details']}\n\n"
        output += "--- Comparison Chart Generated ---\n"

        output_text_widget.insert(tk.END, output)
        
        # Automatically trigger comparison plot
        compare_similar_medicines(found_medicine['Symptom'])
        
    else:
        output_text_widget.insert(tk.END, f"[ERROR] '{medicine_name}' not found in the database. Please check spelling.")
    
    output_text_widget.config(state=tk.DISABLED) # Make text read-only after insertion


def compare_similar_medicines(symptom_name):
    """
    Finds and plots medicines used for the same symptom, comparing Efficacy vs. Cost.
    This version uses highly dynamic limits to ensure all points are visible.
    """
    comparison_data = []
    
    # Data Gathering: Collect all medicines matching the symptom
    for medicine in MEDICINE_DATA:
        if medicine['Symptom'].lower() == symptom_name.lower():
            comparison_data.append(medicine)
    
    if len(comparison_data) < 2:
        # If less than two medicines, we can't compare, so just exit
        return

    # Prepare data for plotting
    names = [d['Name'] for d in comparison_data]
    efficacies = [d['Efficacy'] for d in comparison_data]
    costs = [d['Cost_per_dose'] for d in comparison_data]

    # --- Debugging check (optional, but confirms data is collected) ---
    # print(f"Plotting {len(names)} medicines: {names}") 
    # -----------------------------------------------------------------

    # Create the comparison graph
    plt.figure(figsize=(8, 6)) 
    
    # Scatter plot: Cost (X) vs Efficacy (Y)
    plt.scatter(costs, efficacies, s=300, alpha=0.7, edgecolors='k', linewidths=1.5)

    # Adding labels next to the points
    for i, name in enumerate(names):
        # Annotate each point with the medicine's name
        plt.annotate(name, (costs[i] + 0.02, efficacies[i] - 0.05), fontsize=9, weight='bold')

    # Setting plot labels and title
    plt.xlabel("Estimated Cost per Dose ($)", fontsize=11)
    plt.ylabel("Efficacy Rating (1-10)", fontsize=11)
    plt.title(f"Comparison: {symptom_name} Medicines", fontsize=13, weight='bold')
    
    # --- FIX: Dynamic Limits with increased buffer for guaranteed visibility ---
    if efficacies:
        min_eff = np.min(efficacies)
        max_eff = np.max(efficacies)
        
        # Set Y-axis (Efficacy) range: Use a generous buffer (1.0) below the minimum.
        # This ensures points near the bottom (like Aspirin at 8.0) and their labels are visible.
        plt.ylim(max(0.0, min_eff - 1.0), max_eff + 0.5) 
    
    if costs:
        min_cost = np.min(costs)
        max_cost = np.max(costs)
        
        # Set X-axis (Cost) range: Use a 10% buffer around the min/max costs.
        plt.xlim(min_cost * 0.9, max_cost * 1.1)
    # ------------------------------------------------------------------------
    
    plt.grid(True, linestyle='--', alpha=0.6)
    plt.show()


# ----------------------------------------------------------------------
# 3. GUI SETUP (Tkinter)
# ----------------------------------------------------------------------

def setup_gui():
    """Sets up the main Tkinter window and widgets."""
    
    # 1. Initialize the main window
    root = tk.Tk()
    root.title("Medicine Enquiry System - GUI")
    
    # 2. Configure the main frame for layout
    main_frame = ttk.Frame(root, padding="15 15 15 15")
    main_frame.grid(column=0, row=0, sticky=(tk.W, tk.E, tk.N, tk.S))
    root.grid_columnconfigure(0, weight=1)
    root.grid_rowconfigure(0, weight=1)

    # 3. Title Label
    ttk.Label(main_frame, text="Medicine Enquiry System", font=("Arial", 16, "bold")).grid(
        column=0, row=0, columnspan=2, pady=(0, 15))

    # 4. Input Area
    ttk.Label(main_frame, text="Enter Medicine Name:").grid(column=0, row=1, sticky=tk.W, pady=5)
    
    # Variable to hold the user's input text
    med_name_var = tk.StringVar()
    med_name_entry = ttk.Entry(main_frame, width=30, textvariable=med_name_var)
    med_name_entry.grid(column=1, row=1, sticky=(tk.W, tk.E), padx=10)
    med_name_entry.focus() # Automatically set focus to the input box

    # 5. Search Button
    search_button = ttk.Button(main_frame, text="Search & Compare",
                               command=lambda: get_medicine_details(med_name_var.get(), output_text))
    search_button.grid(column=0, row=2, columnspan=2, pady=(10, 15))

    # 6. Output Area (Scrolled Text Widget for details and errors)
    ttk.Label(main_frame, text="Results:").grid(column=0, row=3, sticky=tk.W, pady=5)
    
    # ScrolledText widget allows us to display multi-line text with a scrollbar
    output_text = scrolledtext.ScrolledText(main_frame, wrap=tk.WORD, width=50, height=15, 
                                            font=("Arial", 10), padx=5, pady=5)
    output_text.grid(column=0, row=4, columnspan=2, sticky=(tk.W, tk.E))
    output_text.config(state=tk.DISABLED) # Initially set to read-only
    
    # 7. Start the main GUI loop
    root.mainloop()

# ----------------------------------------------------------------------
# 4. MAIN EXECUTION
# ----------------------------------------------------------------------

if __name__ == "__main__":
    # First, try to load the data
    if load_data_from_csv():
        # If data loaded successfully, start the GUI
        setup_gui()
    # If data failed to load, the load_data_from_csv function now displays a pop-up error.
