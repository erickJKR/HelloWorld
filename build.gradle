from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import  expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait

#Importar librería para usar mongo
from pymongo import MongoClient

#Ubicación del servidor de Mongo
client = MongoClient('localhost')
#La base de datos que queremos utilizar, si no existe se crea
db = client['anuncios']
#Lo que queremos guardar va dentro de una colección
col = db['olxmercado']

#Instancia del driver de Google Chrome
driver = webdriver.Chrome('./chromedriver')

#Abrir la página donde quiero hacer scrapy
driver.get('https://www.olx.com.ec/items/q-abono')

for i in range(4):
    try:
        #Espera por eventos
        #Va a esperar hasta 10s que el XPATH se encuentre lleno
        boton = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.XPATH, '//button[@data-aut-id="btnLoadMore"]'))
        )

        boton.click()

        #Tengo que esperar que la información nueva se cargue
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located(By.XPATH, '//li[@data-aut-id="itemBox"]//span[@data-aut-id="itemPrice"]')
        )

    except  Exception as e:
        print(e)
        continue #Rompe las iteracciones

#Encontrar todos los items
#Todos los aununcios en una lista
anuncios = driver.find_elements_by_xpath('//li[@data-aut-id="itemBox"]')

#Extracción de cada uno de los elementos de la lista
for anuncio in anuncios:
    try:
        precio = anuncio.find_element_by_xpath('.//span[@data-aut-id="itemPrice"]').text
        #print(precio)
        titulo = anuncio.find_element_by_xpath('.//span[@data-aut-id="itemTitle"]').text
        #print(titulo)
        ubicacion_completa = anuncio.find_element_by_xpath('.//span[@data-aut-id="item-location"]').text
        ubicacion = ubicacion_completa.split(",")[-1]
        #print(ubicacion)

        #Insertando como que son los primeros datos
        '''
        col.insert_one({
            'titulo': titulo.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
            'precio': precio.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
            'origen': "olx"
        })'''
        #Actualizando los datos
        col.update_one({
            "titulo": titulo.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip()
        },{
            "$set": {
                "titulo": titulo.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
                "precio": precio.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
                "ubicacion": ubicacion.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
                "vendidos": "0",
                "origen": "olx"
            }

        }, upsert=True)

    except:
        continue
