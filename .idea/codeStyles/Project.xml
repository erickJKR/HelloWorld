from scrapy.linkextractors import LinkExtractor
from scrapy.crawler import CrawlerProcess
from scrapy.spiders import CrawlSpider, Rule

#Importar librería para usar mongo
from pymongo import MongoClient

#Ubicación del servidor de Mongo
client = MongoClient('localhost')
#La base de datos que queremos utilizar, si no existe se crea
db = client['anuncios']
#Lo que queremos guardar va dentro de una colección
col = db['olxmercado']

class ExtractorFertilizantes(CrawlSpider):
    contador = 0
    name = "FERTILIZANTE"
    custom_settings = {
        'USER_AGENT': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36',
        'CLOSESPIDER_PAGECOUNT': 200, #El CrawlSpider se cierre luego de visitar n páginas
        'LOG_ENABLED': False # Elimina los miles de logs que salen al ejecutar Scrapy en terminal
    }

    download_delay = 1

    start_urls = ['https://listado.mercadolibre.com.ec/fertilizantes-agricolas#D[A:fertilizantes%20agricolas]']

    # Tupla de reglas
    rules = (
        Rule( # REGLA #1 => HORIZONTALIDAD POR PAGINACION
            LinkExtractor(
                allow=r'/_Desde_\d+' # Patron en donde se utiliza "\d+", expresion que puede tomar el valor de cualquier combinacion de numeros
            ), follow=True),
        Rule( # REGLA #2 => VERTICALIDAD AL DETALLE DE LOS PRODUCTOS
            LinkExtractor(
                allow=r'/MEC-'
            ), follow=True, callback='parse_items'), # Al entrar al detalle de los productos, se llama al callback con la respuesta al requerimiento
    )

    def parse_items(self, response):

        titulo = response.xpath('//h1/text()').get()
        precio = response.xpath('//span[@class="price-tag-fraction"]/text()').get()
        try:
            ubicacion_completa = response.xpath('//*[@id="root-app"]/div/div[3]/div/div[2]/div[2]/div/div/div/div[1]/div/p/text()').get()
            ubicacion = ubicacion_completa.strip().split(",")[0]
        except:
            ubicacion = 'Ninguna'

        try:
            vendidos_cadena = response.xpath('//span[@class="ui-pdp-subtitle"]/text()').get()
            vendidos_arreglo = vendidos_cadena.split(" ")
            vendidos = vendidos_arreglo[4]
        except:
            vendidos = '0'
        '''
                print(titulo)
        print(precio)
        print(ubicacion)
        print(vendidos)
        print(self.contador)
        self.contador = self.contador + 1
        print('*************')
        '''
        # Actualizando los datos
        col.update_one({
            "titulo": titulo.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip()
        }, {
            "$set": {
                "titulo": titulo.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
                "precio": precio.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
                "ubicacion": ubicacion.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
                "vendidos": vendidos.replace("$", "").replace('\n', '').replace('\r', '').replace('\t', '').strip(),
                "origen": "mercado"
            }

        }, upsert=True)


process = CrawlerProcess()
process.crawl(ExtractorFertilizantes)
process.start()

