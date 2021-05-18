import random
from time import sleep
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from Utilidades import Eventos
from Utilidades import Limpieza

#Importar librería para usar mongou
from pymongo import MongoClient

#Ubicación del servidor de Mongo
client = MongoClient('localhost')
#La base de datos que queremos utilizar, si no existe se crea
db = client['anuncios']
#Lo que queremos guardar va dentro de una colección
col = db['alibaba']

#Lista de links
links = [
         ["https://spanish.alibaba.com/products/fertilizantes/CID11502.html?IndexArea=product_en", "Orgánico"],
         ["https://spanish.alibaba.com/products/fertilizantes/CID11504.html?IndexArea=product_en", "Nitrógeno"],
         ["https://spanish.alibaba.com/products/fertilizantes/CID11506.html?IndexArea=product_en", "Potasio"],
         ["https://spanish.alibaba.com/products/fertilizantes/CID11505.html?IndexArea=product_en", "Fosfato"],
         ["https://spanish.alibaba.com/products/fertilizantes/CID11503.html?IndexArea=product_en", "Compuesto"],
         ["https://spanish.alibaba.com/products/fertilizantes/CID100006915.html?IndexArea=product_en", "Urea"],
         ["https://spanish.alibaba.com/products/fertilizantes/CID100003953.html?IndexArea=product_en", "Biológico"],
         ["https://spanish.alibaba.com/products/fertilizantes/CID11599.html?IndexArea=product_en", "Otros"]
        ]

opts = Options()
opts.add_argument(
    "user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/71.0.3578.80 Chrome/71.0.3578.80 Safari/537.36"
)

driver = webdriver.Chrome('./chromedriver.exe', chrome_options=opts)

#Intancia de la clase propia Eventos
evento = Eventos(driver)
limpiar = Limpieza()

#Elemento que hay que esperar
elemento = '//div[@class="list-no-v2-outter J-offer-wrapper"]//a[@class="elements-title-normal one-line"]'

#Total productos
total = 0

for link in links:

    # Url semilla
    driver.get(link[0])

    paginas = 1
    total_paginas = 3
    while paginas <= total_paginas:
        print("############################ ", link[1], "############################ ")
        print("############################ ", paginas, "############################ ")

        #Tengo que esperar que la información nueva se cargue
        evento.esperar_elemento(elemento)

        # Hacemos scroll para tratar de cargar todos los datos
        driver.execute_script("window.scrollBy (0,12000)");

        #Tengo que esperar que la información nueva se cargue
        evento.esperar_elemento(elemento)

        driver.execute_script("window.scrollBy (0,12000)");

        #Tengo que esperar que la información nueva se cargue
        evento.esperar_elemento(elemento)

        productos = driver.find_elements_by_xpath('//div[contains(@class, "J-offer-wrapper")]')
        for producto in productos:
            try:
                titulo = limpiar.limpiar_texto(producto.find_element_by_xpath('.//p[@class="elements-title-normal__content large"]').text)
                precios = limpiar.limpiar_texto(producto.find_element_by_xpath('.//span[@class="elements-offer-price-normal__price"]').text)
                arreglo_precio = precios.split("-")
                precio_inferior = float(arreglo_precio[0])
                precio_superior = float(arreglo_precio[-1])
                cantidad_completa = producto.find_element_by_xpath('.//span[@class="element-offer-minorder-normal__value"]').text
                cantidad = cantidad_completa.strip().split(" ")[0]
                medida = cantidad_completa.strip().split(" ")[1]
                print("titulo: ", titulo)
                print("precio inferior: ", precio_inferior)
                print("precio superior: ", precio_superior)
                print("cantidad: ", cantidad)
                print("medida: ", medida)
                print("categoria: ", link[1])

                # Actualizando los datos
                col.update_one({
                    "titulo": titulo
                }, {
                    "$set": {
                        "titulo": titulo,
                        "precio_inferior": precio_inferior,
                        "precio_superior": precio_superior,
                        "cantidad": cantidad,
                        "medida": medida,
                        "categoria": link[1],
                        "origen": "alibaba",
                        "pais": "China"
                    }

                }, upsert=True)
                total += 1
            except:
                print('Tenemos un error')
                continue

        boton = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.XPATH, '//a[@class="seb-pagination__pages-link pages-next"]'))
        )
        try:
            boton.click()
            sleep(random.uniform(2.0, 3.0))
        except:
            paginas = total_paginas
            continue
        finally:
            paginas += 1

print("###################################: ", total)


