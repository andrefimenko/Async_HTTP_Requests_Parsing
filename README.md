# Async_HTTP_Requests_Parsing
## Нас интересуют некоторые бренды на ОЗОН и мы хотим периодически отслеживать ассортимент и цены на них. Для этих целей написали небольшой скрипт https://pastebin.com/RXynXwJS . Каким образом можно ускорить работу данного скрипта? *
Для того что бы ускорить работу кода делающего много медленных действий нужно запустить их параллельно. Для этого в Питоне есть варианты мультипроцессинг и мультитрединг. Для http запросов целесообразней всего использовать частный случай мультитрединга - асинхронные http-запросы. Воспользуемся библиотекой aiohttp, при помощи которой модуль asyncio будет параллельно запускать корутины делающие http-запросы. А при помощи list comprehension создаём сами корутины.

# import requests
import asyncio
import aiohttp
import time


# Вместо requests будем использовать асинхронный клиент-сервер
async def get_page(category: str, page_id: int, session: aiohttp.ClientSession) -> str:
    if page_id:
        url = 'https://www.ozon.ru/brand/{0}/?page={1}'.format(category, page_id)
    else:
        url = 'https://www.ozon.ru/brand/{0}/'.format(category)
    print('get url: {0}'.format(url))
    async with session.get(url) as response:
        return await response.text()


async def load_data():
    category_list = ['adidas-144082850', 'puma-87235756']
    async with aiohttp.ClientSession() as session:
        for category in category_list:
            task_list = [asyncio.create_task(get_page(category, page_id, session)) for page_id in range(50)]
            # Создаём корутины при помощи list comprehension 
            text = await asyncio.gather(*task_list)
            # обрабатываем полученный текст, сохраняем в файл/базу


if __name__ == '__main__':
    t0 = time.time()
    asyncio.run(load_data())
    print(time.time() - t0)
