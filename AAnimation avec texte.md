###CODE SIMULATION CHUTE EN ESCALADE

#importation des différentes bibliothèques nécessaires pour le code
import numpy as np
import matplotlib.pyplot as plt    # Pour faire des graphiques
import tkinter as tk    # Pour créer une interface graphique (fenêtres, boutons, etc.)
from tkinter import ttk    
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg    #Pour mettre les résultats de matplotlib sur la fenêtre graphique
import matplotlib.animation as animation     #Permet de faire des animations

# --- Création de la Fenêtre principale ---
root = tk.Tk()     # Créer une machine permettant de supporter différents graphiques 
root.title("Simulation de chute en escalade")     #Titre de la fenêtre

# --- Cadre de contrôle gris (en haut) ---
frame_controls = tk.Frame(root) # affichage du cadre 
frame_controls.pack(side=tk.TOP, pady=10) # affichage du cadre

    #Ajout dans le cadre d'un menu ou l'utilisateur choisit la masse du grimpeur

tk.Label(frame_controls, text='Masse du grimpeur (kg)').pack()    # Titre de la boite "spinbox" 
choixmasse = tk.Spinbox(frame_controls, from_=50, to=150, increment=0.5, width=5)    #choix de la masse avec un incrément de 0.5kg / dimension et placement de la boite 
choixmasse.pack() # disposition de la boite dans la fenetre 

    # Ajout dans le cadre d'un menu où l'utilisateur choisit la longueur de la corde
tk.Label(frame_controls, text='Longueur de la corde à partir de la dernière dégaine (m)').pack()     # Titre de la boite "spinbox" 
longueurcorde = tk.Spinbox(frame_controls, from_=2, to=50, increment=0.5, width=5)    #choix de la longueur de la corde à partir de la dégaine avec un incrément de 0.5m / dimension et placement de la boite 
longueurcorde.pack()

    # Ajout dans le cadre d'un menu où l'utilisateur choisit la longueur du mou donné au grimpeur lors de sa chute
tk.Label(frame_controls, text='Corde supplémentaire/mou (m)').pack()    # Titre de la boite "spinbox" 
slack_entry = tk.Entry(frame_controls)    #Entrée par l'utilisateur de la longueur du mou avec le clavier, avec un incrément de 0.5m 
slack_entry.insert(0, "0")    #Valeur par défaut 
slack_entry.pack()    # Disposition dans le cadre de control 

# --- Cadres pour affichage des graphiques ---
frame_left = tk.Frame(root)     #Créer des cadres pour les graphique dans l'interface sur la gauche de celle-ci
frame_left.pack(side=tk.LEFT, fill=tk.BOTH, expand=1)     

frame_right = tk.Frame(root)    #Créer des cadres pour les graphique dans l'interface sur la droite de celle-ci
frame_right.pack(side=tk.RIGHT, fill=tk.BOTH, expand=1)

# --- Graphiques matplotlib ---
#graphique de la position
fig_pos, ax_pos = plt.subplots()     #Tracer les axes du graphique
canvas_pos = FigureCanvasTkAgg(fig_pos, master=frame_left)    #Convertir les résultats en widgets Tkinter
canvas_pos.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)    #Mettre le graphique sur la fenêtre Tkinter

#graphique de la vitesse
fig_vitesse, ax_vitesse = plt.subplots()    #Tracer les axes du graphique
canvas_vitesse = FigureCanvasTkAgg(fig_vitesse, master=frame_left)    #Convertir les résultats en widgets Tkinter
canvas_vitesse.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)     #Mettre le graphique sur la fenêtre Tkinter

#cadre de l'animation
fig_anim, ax_anim = plt.subplots()    #Tracer les axes du graphique
canvas_anim = FigureCanvasTkAgg(fig_anim, master=root)    #Convertir les résultats en widgets Tkinter principal
canvas_anim.get_tk_widget().pack(side=tk.LEFT, fill=tk.BOTH, expand=1)    # #Mettre le graphique sur la fenêtre Tkinter

#graphique de la force
fig_force, ax_force = plt.subplots()    #Tracer les axes du graphique
canvas_force = FigureCanvasTkAgg(fig_force, master=frame_right)    #Convertir les résultats en widgets Tkinter principal
canvas_force.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)    #Mettre le graphique sur la fenêtre Tkinter

# Charger l'image de fond
background_image = plt.imread('mur_escalade.jpg')    #Permet de mettre un foud d'écran à l'animation

# --- Fonction principale ---
def start_animation(ax_pos, canvas_pos, ax_vitesse, canvas_vitesse, ax_anim, ax_force, canvas_force, canvas_anim, fig_anim):

    longueur_corde = float(longueurcorde.get())    #Récupère les données rentrées par l'utilisateur
    poids_grimpeur = float(choixmasse.get())
    slack = float(slack_entry.get())

    #Constantes
    m = poids_grimpeur
    g = 9.81
    L0 = longueur_corde
    
   
    dt = 0.005    #Pas de temps
    Tmax = 5    #Durée totale de l'animation

    def simulate_chute(L0_effectif, mouvement_point_fixe=None, ):    #fonction qui permet de déterminer à chaque instant la position, la vitesse et les forces appliquées sur le grimpeur
        #initialisation des données, on considère le grimpeur comme un point fixe avant sa chute

        k = 78000/(L0_effectif + 10)   #Constante de raideur de la corde
        b = 2 * np.sqrt(m * k)    #Coef d'amortissement en régime critique
        
        y = 0    
        v = 0
        t = 0
        pos, vit, temps, force = [], [], [], []    #listes qui stockent les données calculées, initialement vides 

        while t < Tmax:    # tant que le temps est inférieur à la durée totale de la simulation :
            y_ancrage = mouvement_point_fixe(t, L0) if mouvement_point_fixe else 0    #Place le point d'ancrage sur les assureurs (fixes ou mobiles)
            extension = y - y_ancrage - L0_effectif    #Extension de la corde par rapport à sa longueur à vide
            F_spring = -k * extension if extension > 0 else 0    #Force de rappel de la corde
            F_damping = -b * v if extension > 0 else 0    #Force d'amortissement modélisée par un frottement fluide
            a = (m * g + F_spring + F_damping) / m    #Accélération calculée grâce au PFD
            v += a * dt    #Vitesse calculée par la méthode d'Euler
            y += v * dt    #Position calculée par la méthode d'Euler
            t += dt    

            #Les données calculées par les fonctions sont mises dans les listes correspondantes 
            pos.append(y - y_ancrage)
            vit.append(v)
            temps.append(t)
            force.append(m * g - F_spring - F_damping)    # la force subie par le grimpeur est la somme de son poids et des forces de rappel de la corde

        return temps, pos, vit, force    # nous avons maintenant accès à ces données sous forme de listes

    def mouvement_assureur(t, L0):    # modélise le saut que ferait l'assureur en freinant la chute de son camarade
        t0 = (L0 / g) ** 0.5 / 2  #Instant où l'assureur doit sauter, légèrement plus tôt que le moment où la corde se tend
        if t > t0 and t < 3 * t0:    #Saut durant toute la période de freinage
            return -2 * (t - t0)    #Il se déplace de 2 m/s
        if t >= 3 * t0:    #arrête le mouvement à la fin du freinage
            return -4 * t0    #Sa position reste constante
        else:
            return 0    #Il reste au sol avant le freinage

    temps_a, positions_a, vitesses_a, force_a = simulate_chute(L0)    #valeurs calculées lorsqu'il n'y pas de mou 
    temps_b, positions_b, vitesses_b, force_b = simulate_chute(L0 + slack)    #  valeurs calculées lorsqu'il y a du mou qui est donné par l'assureur  
    temps_c, positions_c, vitesses_c, force_c = simulate_chute(L0, mouvement_assureur)    #valeurs calculées lorsque l'assureur saute 

    # --- Graphiques ---
    # graphiques des positions en fonction du temps, les 3 listes de positions sont tracées sur le même graphique
 
    ax_pos.clear()    # remise à 0 initiale
    ax_pos.plot(temps_a, positions_a, label="Sans mou", color='blue')    # position sans mou, dessiné en bleu
    ax_pos.plot(temps_b, positions_b, label="Avec mou", color='green')    # position avec du mou, dessiné en vert
    ax_pos.plot(temps_c, positions_c, label="Assureur saute", color='red')     # position avec le saut de l'assureur, dessiné en rouge
    ax_pos.set_xlabel("Temps (s)")    # dénomination des axes
    ax_pos.set_ylabel("Position (m)")
    ax_pos.legend()    #Place le nom des axes
    ax_pos.set_ylim(-1, L0 + slack + 2)    # valeurs limites pour l'axe vertical
    canvas_pos.draw()    # tracé du graphe


    #graphique des vitesses en fonction du temps, les 3 listes de vitesse sont tracées sur le même graphique
 
    ax_vitesse.clear()    # remise à 0 initiale
    ax_vitesse.plot(temps_a, vitesses_a, label="Sans mou", color='blue')    # vitesse sans mou, dessiné en bleu
    ax_vitesse.plot(temps_b, vitesses_b, label="Avec mou", color='green')    # vitesse avec du mou, dessiné en vert
    ax_vitesse.plot(temps_c, vitesses_c, label="Assureur saute", color='red')    # vitesse avec le saut de l'assureur, dessiné en rouge
    ax_vitesse.set_xlabel("Temps (s)")    # dénomination des axes
    ax_vitesse.set_ylabel("Vitesse (m/s)")
    ax_vitesse.legend()     #Place le nom des axes
    ax_vitesse.set_ylim(min(min(vitesses_a), min(vitesses_b), min(vitesses_c)) - 1,
                        max(max(vitesses_a), max(vitesses_b), max(vitesses_c)) + 1)     #valeurs limites pour l'axe vertical
    canvas_vitesse.draw()    # tracé du graphe

    # graphique des forces subies par le grimpeur en fonction du temps, les 3 listes de forces sont tracées sur le même graphique 
  
    ax_force.clear()    # remise à 0 initiale
    ax_force.plot(temps_a, force_a, label="Sans mou", color='blue')    # forces sans mou, dessiné en bleu
    ax_force.plot(temps_b, force_b, label="Avec mou", color='green')    # forces avec du mou, dessiné en vert
    ax_force.plot(temps_c, force_c, label="Assureur saute", color='red')    # forces avec le saut de l'assureur, dessiné en rouge
    ax_force.set_xlabel("Temps (s)")    # dénomination des axes
    ax_force.set_ylabel("Force subie (N)")
    ax_force.legend()    #Place le nom des axes
    ax_force.set_ylim(0, max(max(force_a), max(force_b), max(force_c)) + 100)    #valeurs limites pour l'axe vertical
    canvas_force.draw()    # tracé du graphe

    ## --- Animation ---
    ax_anim.clear()    #Remise à 0 initiale
    ax_anim.set_xlim(1, 9)    #Dimensions du cadre de l'animation
    ax_anim.set_ylim(L0 + 11, -5)    #Dimensions du cadre de l'animation, dépend de L0 pour s'adapter aux données initiales
    ax_anim.imshow(background_image, extent=[1, 9, L0 + 11, -5], aspect='auto')    #Adapter le fond d'écran au cadre et aux données rentrées

    x_sans_mou = 3    #  position horizontale intiale des grimpeurs 
    x_avec_mou = 5
    x_assureur_saut = 7

    # Position des dégaines, à 1 mètre de la position intiale de chute des grimpeurs
    
    # Dégaines pour "Sans mou"
    ax_anim.plot(x_sans_mou, 1, '^', color='black', markersize=8, label="Dégaines")   

    # Dégaines pour "Avec mou"
    ax_anim.plot(x_avec_mou, 1, '^', color='black', markersize=8)

    # Dégaines pour "Assureur saute"
    ax_anim.plot(x_assureur_saut, 1, '^', color='black', markersize=8)

    # Créations des objets "grimpeurs" et Introduction dans la légende 
    grimpeur_sans_mou, = ax_anim.plot([], [], 'bo', markersize=10, label="Sans mou")    #Création du grimpeur sans mou, représenté par un rond bleu
    grimpeur_avec_mou, = ax_anim.plot([], [], 'go', markersize=10, label="Avec mou")    #Création du grimpeur avec mou, représenté par un rond vert
    grimpeur_assureur_saut, = ax_anim.plot([], [], 'ro', markersize=10, label="Assureur saute")    ##Création du grimpeur avec assureur qui saute, représenté par un rond rouge

    # Ajouter des assureurs statiques avec la même légende
        # Leur position dépend de L0 pour s'adapter aux données de l'utilisateur
    assureur_statique1, = ax_anim.plot([x_sans_mou + 1], [L0 + 10], 'ks', markersize=10, label="Assureur")
    assureur_statique2, = ax_anim.plot([x_avec_mou + 1], [L0 + 10], 'ks', markersize=10)
        # Sa position n'est pas encore définie car il sera en mouvement
    assureur_saut, = ax_anim.plot([], [], 'ks', markersize=10)

    ax_anim.legend(loc='lower left', fontsize=9)    #Placement de la légende en bas à droite de l'animation
    
    #Création des cordes en deux segments : de la dégaine au grimpeur, et de la degaine à l'assureur 
    #Les listes vides seront remplies au fur et à mesure de la chute
    #Elles ont une épaisseur "lw" et une couleur (bleu, vert ou rouge)
    corde1_sans_mou, = ax_anim.plot([], [], 'b-', lw=2)    #épaisseur de la corde = lw
    corde2_sans_mou, = ax_anim.plot([], [], 'b-', lw=2)

    corde1_avec_mou, = ax_anim.plot([], [], 'g-', lw=2)
    corde2_avec_mou, = ax_anim.plot([], [], 'g-', lw=2)

    corde1_assureur_saut, = ax_anim.plot([], [], 'r-', lw=2)
    corde2_assureur_saut, = ax_anim.plot([], [], 'r-', lw=2)

    n_frames = len(positions_a)    # nombre total d'images dans l'animation = nombre de valeurs dans nos listes

    # FOnction qui arrête l'animation lorsque le temps défnit est écoulé
    def update(frame):
        if frame >= n_frames:
            return []

        # Positions verticales
        y_a = max(0, positions_a[frame])
        y_b = max(0, positions_b[frame])
        y_c = max(0, positions_c[frame])
        t = frame * dt     # Mettre à jour l'image avec les nouvelles données à un instant t
        y_assureur_c = L0 + 10 + mouvement_assureur(t, L0)    #L'assureur part de sa position initiale (L0 + 10) puis saute selon le programme "mouvement_assureur" écrit plus tôt

        #Positions fixes des assureurs
        y_assureur_a = L0 + 10 
        y_assureur_b = L0 + 10

        #Mise à jour des positions des grimpeurs
        grimpeur_sans_mou.set_data([x_sans_mou], [y_a])
        grimpeur_avec_mou.set_data([x_avec_mou], [y_b])
        grimpeur_assureur_saut.set_data([x_assureur_saut], [y_c])

        # Mise à jour des données de la position de l'assureur qui saute
        assureur_saut.set_data([x_assureur_saut + 1], [y_assureur_c])

        #Mise à jour de la longueur des cordes 
        corde1_sans_mou.set_data([x_sans_mou, x_sans_mou], [y_a, 1])
        corde2_sans_mou.set_data([x_sans_mou + 1, x_sans_mou], [y_assureur_a, 1])

        corde1_avec_mou.set_data([x_avec_mou, x_avec_mou], [y_b, 1])
        corde2_avec_mou.set_data([x_avec_mou + 1, x_avec_mou], [y_assureur_b, 1])

        corde1_assureur_saut.set_data([x_assureur_saut, x_assureur_saut], [y_c, 1])
        corde2_assureur_saut.set_data([x_assureur_saut + 1, x_assureur_saut], [y_assureur_c, 1])

        return (
            grimpeur_sans_mou, grimpeur_avec_mou, grimpeur_assureur_saut,
            assureur_saut,
            corde1_sans_mou, corde2_sans_mou,
            corde1_avec_mou, corde2_avec_mou,
            corde1_assureur_saut, corde2_assureur_saut
        )                                                    #Sortie de la fonction

    ani = animation.FuncAnimation(fig_anim, update, frames=n_frames, interval=5, blit=True)    #Fonction animation qui est gérée par la librairie associée avec les variables définies
    canvas_anim.draw()    #Force le programme a tracer l'animation sur l'interface


# --- Fonction qui indique si le choc est trop sévère pour le grimpeur ---
    danger = any(f > 6000 for f in force_b)    #Si la force subie dépasse 6 kN, le message s'affiche
    message = "Danger, chute douloureuse\n" if danger else "Pas de risque pour le grimpeur\n"    #Contenu du message
    texte.delete(1.0, tk.END)    #Prépare la zone d'affichage en la rendant vierge 
    texte.insert(tk.END, message)    #Place le message dans le cadre de contrôle

    


# --- Zone de message ---
frame_message = ttk.Frame(frame_controls)     # Création cadre de message
frame_message.pack(pady=10, fill='x')    #afficher le cadre de message avec les caractéristiques voulues

texte = tk.Text(frame_message, height=3, width=50, bg="#f7f7f7", fg="#111", font=("Helvetica", 15))
texte.pack()

# --- Bouton lancer la simulation ---
btLancer = tk.Button(frame_controls, text="Lancer la simulation", command=lambda: start_animation(
    ax_pos, canvas_pos, ax_vitesse, canvas_vitesse,
    ax_anim, ax_force, canvas_force, canvas_anim, fig_anim
))    # Appelle la fonction start_animation pour lance la simulation 
btLancer.pack(pady=10)     # position dans la fenetre de controle

# --- Bouton quitter ---
btQuit = tk.Button(root, text='Quitter', command=root.destroy)
btQuit.pack(side=tk.BOTTOM, padx=10, pady=10)

# --- Lancement de l'interface ---
root.mainloop()
