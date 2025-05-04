###CODE AVEC TEXTE DANGER ET ANIMATION BG
import numpy as np
import matplotlib.pyplot as plt
import tkinter as tk
from tkinter import ttk
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import matplotlib.animation as animation

# --- Fenêtre principale ---
root = tk.Tk()
root.title("Simulation de chute en escalade")

# --- Cadre de contrôle (en haut) ---
frame_controls = tk.Frame(root)
frame_controls.pack(side=tk.TOP, pady=10)

tk.Label(frame_controls, text='Masse du grimpeur (kg)').pack()
choixmasse = tk.Spinbox(frame_controls, from_=50, to=150, increment=0.5, width=5)
choixmasse.pack()

tk.Label(frame_controls, text='Longueur de la corde (m)').pack()
longueurcorde = tk.Spinbox(frame_controls, from_=15, to=100, increment=0.5, width=5)
longueurcorde.pack()

tk.Label(frame_controls, text='Corde supplémentaire/mou (m)').pack()
slack_entry = tk.Entry(frame_controls)
slack_entry.insert(0, "0")
slack_entry.pack()

# --- Cadres pour affichage des graphiques ---
frame_left = tk.Frame(root)
frame_left.pack(side=tk.LEFT, fill=tk.BOTH, expand=1)

frame_right = tk.Frame(root)
frame_right.pack(side=tk.RIGHT, fill=tk.BOTH, expand=1)

# --- Graphiques matplotlib ---
fig_pos, ax_pos = plt.subplots()
canvas_pos = FigureCanvasTkAgg(fig_pos, master=frame_left)
canvas_pos.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)

fig_vitesse, ax_vitesse = plt.subplots()
canvas_vitesse = FigureCanvasTkAgg(fig_vitesse, master=frame_left)
canvas_vitesse.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)

fig_anim, ax_anim = plt.subplots()
canvas_anim = FigureCanvasTkAgg(fig_anim, master=root)
canvas_anim.get_tk_widget().pack(side=tk.LEFT, fill=tk.BOTH, expand=1)

fig_force, ax_force = plt.subplots()
canvas_force = FigureCanvasTkAgg(fig_force, master=frame_right)
canvas_force.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)

# Charger l'image de fond
background_image = plt.imread('mur_escalade.jpg')

# --- Fonction principale ---
def start_animation(ax_pos, canvas_pos, ax_vitesse, canvas_vitesse, ax_anim, ax_force, canvas_force, canvas_anim, fig_anim):
    longueur_corde = float(longueurcorde.get())
    poids_grimpeur = float(choixmasse.get())
    slack = float(slack_entry.get())

    m = poids_grimpeur
    g = 9.81
    L0 = longueur_corde
    k = 1500
    b = 2 * np.sqrt(m * k)
    dt = 0.005
    Tmax = 5
    saut_assureur = 1

    def simulate_chute(L0_effectif, mouvement_point_fixe=None):
        y = 0
        v = 0
        t = 0
        pos, vit, temps, force = [], [], [], []

        while t < Tmax:
            y_ancrage = mouvement_point_fixe(t) if mouvement_point_fixe else 0
            extension = y - y_ancrage - L0_effectif
            F_spring = -k * extension if extension > 0 else 0
            F_damping = -b * v if extension > 0 else 0
            a = (m * g + F_spring + F_damping) / m
            v += a * dt
            y += v * dt
            t += dt

            if y < 0:
                y = 0
                v = 0

            pos.append(y - y_ancrage)
            vit.append(v)
            temps.append(t)
            force.append(m * g - F_spring - F_damping)

        return temps, pos, vit, force

    temps_a, positions_a, vitesses_a, force_a = simulate_chute(L0)
    temps_b, positions_b, vitesses_b, force_b = simulate_chute(L0 + slack)

    def mouvement_assureur(t):
        # L'assureur monte progressivement pendant les 0.5 premières secondes
        if t < 1:
            return -saut_assureur * t / 1
        else:
            return -saut_assureur

    temps_c, positions_c, vitesses_c, force_c = simulate_chute(L0, mouvement_assureur)

    # --- Graphiques statiques ---
    ax_pos.clear()
    ax_pos.plot(temps_a, positions_a, label="Sans mou", color='blue')
    ax_pos.plot(temps_b, positions_b, label="Avec mou", color='green')
    ax_pos.plot(temps_c, positions_c, label="Assureur saute", color='red')
    ax_pos.set_xlabel("Temps (s)")
    ax_pos.set_ylabel("Position (m)")
    ax_pos.legend()
    ax_pos.set_ylim(-10, L0 + slack + 2)
    canvas_pos.draw()

    ax_vitesse.clear()
    ax_vitesse.plot(temps_a, vitesses_a, label="Sans mou", color='blue')
    ax_vitesse.plot(temps_b, vitesses_b, label="Avec mou", color='green')
    ax_vitesse.plot(temps_c, vitesses_c, label="Assureur saute", color='red')
    ax_vitesse.set_xlabel("Temps (s)")
    ax_vitesse.set_ylabel("Vitesse (m/s)")
    ax_vitesse.legend()
    ax_vitesse.set_ylim(min(min(vitesses_a), min(vitesses_b), min(vitesses_c)) - 1,
                        max(max(vitesses_a), max(vitesses_b), max(vitesses_c)) + 1)
    canvas_vitesse.draw()

    ax_force.clear()
    ax_force.plot(temps_a, force_a, label="Sans mou", color='blue')
    ax_force.plot(temps_b, force_b, label="Avec mou", color='green')
    ax_force.plot(temps_c, force_c, label="Assureur saute", color='red')
    ax_force.set_xlabel("Temps (s)")
    ax_force.set_ylabel("Force subie (N)")
    ax_force.legend()
    ax_force.set_ylim(0, max(max(force_a), max(force_b), max(force_c)) + 100)
    canvas_force.draw()

    # --- Animation ---
    ax_anim.clear()
    ax_anim.set_xlim(1, 9)
    ax_anim.set_ylim(L0 + slack + 2, -2)

    # Image de fond corrigée
    ax_anim.imshow(background_image, extent=[1, 9, L0 + slack + 2, -2], aspect='auto')

    x_sans_mou = 3
    x_avec_mou = 5
    x_assureur_saut = 7

    # Dégaines fixes principales (en haut)
    ax_anim.plot([x_sans_mou, x_avec_mou, x_assureur_saut], [0, 0, 0], '^', color='black', markersize=10, label="Dégaine")

    # Dégaines intermédiaires supplémentaires
    degaine_y1 = (L0 + slack) / 3
    degaine_y2 = 2 * (L0 + slack) / 3

    # Ajout des dégaines intermédiaires pour chaque scénario
    # Ajout des dégaines intermédiaires uniquement pour "avec mou"
    ax_anim.plot(x_avec_mou, degaine_y1, '^', color='black', markersize=8)
    ax_anim.plot(x_avec_mou, degaine_y2, '^', color='black', markersize=8)

    # Objets dynamiques
    grimpeur_sans_mou, = ax_anim.plot([], [], 'bo', markersize=10, label="Sans mou")
    grimpeur_avec_mou, = ax_anim.plot([], [], 'go', markersize=10, label="Avec mou")
    grimpeur_assureur_saut, = ax_anim.plot([], [], 'ro', markersize=10, label="Assureur saute")

    assureur_sans_mou, = ax_anim.plot([x_sans_mou], [L0], 'ks', markersize=10)
    assureur_avec_mou, = ax_anim.plot([x_avec_mou], [L0 + slack], 'ks', markersize=10)
    assureur_saut, = ax_anim.plot([], [], 'ks', markersize=10, label="Assureur")

    corde1_sans_mou, = ax_anim.plot([], [], 'b-', lw=2)
    corde2_sans_mou, = ax_anim.plot([], [], 'b-', lw=2)

    corde1_avec_mou, = ax_anim.plot([], [], 'g-', lw=2)
    corde2_avec_mou, = ax_anim.plot([], [], 'g-', lw=2)

    corde1_assureur_saut, = ax_anim.plot([], [], 'r-', lw=2)
    corde2_assureur_saut, = ax_anim.plot([], [], 'r-', lw=2)

    ax_anim.legend(fontsize=10)

    n_frames = len(positions_a)

    def update(frame):
        if frame >= n_frames:
            return []

        y_a = positions_a[frame]
        y_b = positions_b[frame]
        y_c = positions_c[frame]

        # Calcul de la position de l'assureur qui saute
        t = frame * dt
        y_assureur_c = L0 + mouvement_assureur(t)

        # Sécurité
        y_a = max(0, y_a)
        y_b = max(0, y_b)
        y_c = max(0, y_c)

        grimpeur_sans_mou.set_data([x_sans_mou], [y_a])
        grimpeur_avec_mou.set_data([x_avec_mou], [y_b])
        grimpeur_assureur_saut.set_data([x_assureur_saut], [y_c])

        assureur_saut.set_data([x_assureur_saut], [y_assureur_c])

        corde1_sans_mou.set_data([x_sans_mou, x_sans_mou], [y_a, 0])
        corde2_sans_mou.set_data([x_sans_mou, x_sans_mou], [0, L0])

        corde1_avec_mou.set_data([x_avec_mou, x_avec_mou], [y_b, 0])
        corde2_avec_mou.set_data([x_avec_mou, x_avec_mou], [0, L0 + slack])

        corde1_assureur_saut.set_data([x_assureur_saut, x_assureur_saut], [y_c, 0])
        corde2_assureur_saut.set_data([x_assureur_saut, x_assureur_saut], [0, y_assureur_c])

        return (
            grimpeur_sans_mou, grimpeur_avec_mou, grimpeur_assureur_saut,
            assureur_saut,
            corde1_sans_mou, corde2_sans_mou,
            corde1_avec_mou, corde2_avec_mou,
            corde1_assureur_saut, corde2_assureur_saut
        )

    # Vérification du danger
    danger = any(f > 14000 for f in force_b)
    message = ""
    if danger:
        message = "⚠️ Danger, risque de fracture pour le grimpeur (avec mou)\n"
    else:
        message = "✅ Pas de risque majeur pour le grimpeur (avec mou)\n"
    
    # Mise à jour du texte de danger
    texte.delete(1.0, tk.END)
    texte.insert(tk.END, message)

    ani = animation.FuncAnimation(fig_anim, update, frames=n_frames, interval=5, blit=True)
    canvas_anim.draw()

# --- Zone de message ---
frame_message = ttk.Frame(frame_controls)
frame_message.pack(pady=10, fill='x')

texte = tk.Text(frame_message, height=6, width=70, bg="#f7f7f7", fg="#111", font=("Helvetica", 10))
texte.pack()

# --- Bouton lancer la simulation ---
btLancer = tk.Button(frame_controls, text="Lancer la simulation", command=lambda: start_animation(
    ax_pos, canvas_pos, ax_vitesse, canvas_vitesse,
    ax_anim, ax_force, canvas_force, canvas_anim, fig_anim
))
btLancer.pack(pady=10)

# --- Bouton quitter ---
btQuit = tk.Button(root, text='Quitter', command=root.destroy)
btQuit.pack(side=tk.BOTTOM, padx=10, pady=10)

# --- Lancement de l'interface ---
root.mainloop()
