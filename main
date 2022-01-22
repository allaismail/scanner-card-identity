from tkinter import *
from tkinter import filedialog
from tkinter import messagebox
import mysql.connector as MC
from random import *
from PIL import ImageTk,Image
import pytesseract
import cv2
import numpy as np
per = 10
imgQ = cv2.imread('.//images/image_standard.jpg')
h, w, c = imgQ.shape
# cette fonction permet de redementioner la'image et
def traitement_img(myimg) :
    orb = cv2.ORB_create(5000)
    kp1, des1 = orb.detectAndCompute(imgQ, None)
    kp2, des2 = orb.detectAndCompute(myimg, None)
    bf = cv2.BFMatcher(cv2.NORM_HAMMING)
    matches = bf.match(des2, des1)
    matches.sort(key=lambda x: x.distance)
    good = matches[:int(len(matches) * (per / 100))]
    imgMatch = cv2.drawMatches(myimg, kp2, imgQ, kp1, good, None, flags=2)
    srcPoints = np.float32([kp2[m.queryIdx].pt for m in good]).reshape(-1, 1, 2)
    dstPoints = np.float32([kp1[m.trainIdx].pt for m in good]).reshape(-1, 1, 2)
    M, _ = cv2.findHomography(srcPoints, dstPoints, cv2.RANSAC, 5.8)
    imgscan = cv2.warpPerspective(myimg, M, (w, h))
    imgscan = cv2.resize(imgscan, (w, h))
    return imgscan

# cette fonction permet de transformer l'image a une image grer
def grayscale(image):
    return cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
# detection des visages dans une image
def detection_image(gray):
    class_Cascade = cv2.CascadeClassifier(".//images/haarcascade_frontalface_alt2.xml") # model d'entinement sur les visage d'humain
    faces = class_Cascade.detectMultiScale( gray, scaleFactor=1.1, minNeighbors=5,minSize=(60,60), flags=cv2.CASCADE_SCALE_IMAGE)
    return faces
# Réduction de bruits d'une image
def remove_noise(image):
    return cv2.medianBlur(image, 1)
# Seuillage
def thresholding(image):
    return cv2.threshold(image, 200, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)[1]
# detecter un texte sur  une zone donnee sur un'image
def tere_chaine(image, x, y, w, h):
    region_Nom = image[y:h, x:w]
    region_Nom = remove_noise(thresholding(grayscale(region_Nom)))
    NomCI = pytesseract.image_to_string(region_Nom)
    NomCI = NomCI.replace('\n', '')
    NomCI = NomCI.strip()
    return NomCI
def open():
    # l'image 2
    img2 = ImageTk.PhotoImage(Image.open('.//images/logo.png').resize((319, 166), Image.ANTIALIAS))
    logo2 = Label(app, image=img2)
    logo2.img = img2  # Keep a reference in case this code put is in a function.
    logo2.place(x=316, y=0)  # Place a la bas de l'interface

    imagelien = filedialog.askopenfilename(initialdir="C://Users/alla_ismail/Pictures/scanner", title="choisir l'image",filetypes=(("ex png", "*.png"), ("ex jpg", "*.jpg"),("tous les ex", "*.*")))
    imagecv = cv2.imread(imagelien)
    image = traitement_img(imagecv)
    gray = grayscale(image)
    faces = detection_image(gray)
    #faire un rectangle sur les différentes vissage(s) de l'image
    for (x, y, w, h) in faces:
        cv2.rectangle(image, (x, y), (x + w, y + h), (0, 255, 0), 2)
    #on declarer ces variable globalement pour les utiliser dans la fonction commiter
    global nom,prenom,cin,date,adress,lien
    # retirer les donnnes
    # retirer le prenom
    prenom = tere_chaine(image,5,121,115,154)
    # retirer le nom
    nom = tere_chaine(image,5,175,115,208)
    # retirer la date de naissance
    date = tere_chaine(image,146,200,290,240)
    # retirer le cin
    cin = tere_chaine(image,470,320,600,360)
    # retirer l'adresse en francais
    adress = tere_chaine(image,28,250,220,290)
    # l'image 2 au niveau
    img2 = ImageTk.PhotoImage(Image.open('.//images/logo.png').resize((316, 166), Image.ANTIALIAS))
    logo2 = Label(app, image=img2)
    logo2.img = img2  # Keep a reference in case this code put is in a function.
    logo2.place(x=316, y=0)  # Place a la bas de l'interface
    # les deffirent atributs
    Label(app, text=nom,bg="white", font=("Courier", 13)).place(x=246, y=200)
    Label(app, text=prenom,bg="white", font=("Courier", 13)).place(x=246, y=225)
    Label(app, text=cin,bg="white", font=("Courier", 13)).place(x=248, y=250)
    Label(app, text=date,bg="white", font=("Courier", 13)).place(x=246, y=275)
    Label(app, text=adress,bg="white", font=("Courier", 13)).place(x=246, y=300)
    f=faces[0]
    x = random()
    lien = 'C:/Users/alla_ismail/PycharmProjects/projet/images/resulta' + str(x) + '.png'
    cv2.imwrite(lien, image[f[1]:f[1] + f[3], f[0]:f[0] + f[2]])
    img = ImageTk.PhotoImage(Image.open(lien).resize((160,160), Image.ANTIALIAS) )
    lbl =Label(app, image=img)
    lbl.img = img     # Keep a reference in case this code put is in a function.
    lbl.place(x=470,y=175)  # Place a la bas de l'interface
# base de donne
def commiter():
    global conn, cursor
    try:
        conn = MC.connect(host='localhost', database='projet', user='root', password='')
        cursor = conn.cursor()
        req = 'INSERT INTO carte(cin,nom,prenom, date_nai,adresse,image) VALUES (%s,%s,%s,%s,%s,%s)'
        data=( cin, nom , prenom,date,adress,lien)
        cursor.execute(req,data)
        conn.commit()
        messagebox.showinfo("information", "les données sont enregistrer avec succ ées")
    except MC.Error as err:
        messagebox.showerror("Error", "les données ne sont pas enregistrer \n s'il vous plait verifier la conixion \n de la base de données")
    finally:
        if conn.is_connected():
            cursor.close()
            conn.close()

# déclarer une fenetre
app = Tk()
app.geometry("640x400+300+100") # la dimention de la fenetre
app.title("scanner la carte d'identité") # le titre de notre propre interface graphique
app.resizable(width=0, height=0) # fixer la taille on ne peut pas ici redemention de la fenetre
# l'image 1
img = ImageTk.PhotoImage(Image.open('.//images/centre.png').resize((314,167), Image.ANTIALIAS))
logo =Label(app, image=img)
logo.img = img     # Keep a reference in case this code put is in a function.
logo.place(x=0, y=0)  # Place a la bas de l'interface

# l'image 2
img2 = ImageTk.PhotoImage(Image.open('.//images/logo.png').resize((319,110), Image.ANTIALIAS))
logo2 =Label(app, image=img2)
logo2.img = img2     # Keep a reference in case this code put is in a function.
logo2.place(x=316 , y=0)  # Place a la bas de l'interface

#les canvas

Canvas(app, bg="ivory", width='410',height="-56",bd="110", highlightthickness="5",highlightbackground="sky blue").place(x=0, y=170)
Canvas(app, bg="ivory", width='0',height="164", bd="0", highlightthickness="4",highlightbackground="sky blue").place(x=461, y=170)
Label(app, text="il faut que l'image ", fg="#FF0021", font=("Courier", 11)).place(x=466, y=118)
Label(app, text="à une bon qualité ", fg="#FF0021", font=("Courier", 11)).place(x=470, y=142)
# les differentes attribute
Label(app, text="S'il vous plait vérifier les  informations  :", fg="#AA9988", font=("Courier", 12)).place(x=5,y=175)
Label(app, text="Nom                   :", font=("Courier", 13)).place(x=6, y=200)
Label(app, text="Prenom                :", font=("Courier", 13)).place(x=6, y=225)
Label(app, text="CIN	              :", font=("Courier", 13)).place(x=6, y=250)
Label(app, text="Date de Naissance     :", font=("Courier", 13)).place(x=6, y=275)
Label(app, text="Adresse de Naissance  :", font=("Courier", 13)).place(x=6, y=300)
Button(app, text="valider", bg="green", font=("Courier,12"),width=20,height=2, command=commiter).place(x=442,  y=346)
Button(app, text="la personne suivant", bg="orange", font=("Courier,12"),width=20, height=2, command=open).place(x=224, y=346)
Button(app, text="Quitter", bg="red", font=("Courier,12"), width=20, height=2, command=quit).place(x=6, y=346)
Button(app, text="** choisir l'image **", bg='yellow', width=20, height=3, command=open).place(x=318, y=114)
app.mainloop()
