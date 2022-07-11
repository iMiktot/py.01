## Домашнее задание к лекции 8.«Работа с библиотекой requests, http-запросы»

#### Выполнил: Смирнов Роман

#### Группа: SQLPY-52

### Задача №1

>Кто самый умный супергерой?
>Есть API по информации о супергероях с информацией по всем супергероям. Нужно определить кто самый умный(intelligence) из трех супергероев- Hulk, Captain America, Thanos.

```py
import requests

TOKEN = '2619421814940190'  # константа токена
urls = [
    f'https://www.superheroapi.com/api.php/{TOKEN}/search/Hulk',
    f'https://www.superheroapi.com/api.php/{TOKEN}/search/Thanos',
    f'https://www.superheroapi.com/api.php/{TOKEN}/search/Captain%America',
]  # список адресов


def requests_get(url_all):
    # принимает список адресов
    r = (requests.get(url) for url in url_all)
    return r

def parser():
    # функция парсинга интелекта
    super_man = []
    for item in requests_get(urls):
        intelligence = item.json()
        try:
            for power_stats in intelligence['results']:
                super_man.append({
                    'name': power_stats['name'],
                    'intelligence': power_stats['powerstats']['intelligence'],
                })
        except KeyError:
            print(f"Проверте ссылки urls: {urls}")

    intelligence_super_hero = 0
    name = ''
    for intelligence_hero in super_man:
        if intelligence_super_hero < int(intelligence_hero['intelligence']):
            intelligence_super_hero = int(intelligence_hero['intelligence'])
            name = intelligence_hero['name']

    print(f"Самый интелектуальный {name}, интелект: {intelligence_super_hero}")


parser()
```

### Задача №2

>У Яндекс.Диска есть очень удобное и простое API. Для описания всех его методов существует Полигон. Нужно написать программу, которая принимает на вход путь до файла на компьютере и сохраняет на Яндекс.Диск с таким же именем.

>Все ответы приходят в формате json;
>Загрузка файла по ссылке происходит с помощью метода put и передачи туда данных;
>Токен можно получить кликнув на полигоне на кнопку "Получить OAuth-токен".
>HOST: https://cloud-api.yandex.net:443

```py
from pprint import pprint
import requests

TOKEN = ''


class YandexDisk:

    def __init__(self, token):
        self.token = token

    def get_headers(self):
        return {
            'Content-Type': 'application/json',
            'Authorization': 'OAuth {}'.format(self.token)
        }

    def get_files_list(self):
        files_url = 'https://cloud-api.yandex.net/v1/disk/resources/files'
        headers = self.get_headers()
        response = requests.get(files_url, headers=headers)
        return response.json()

    def _get_upload_link(self, disk_file_path):
        upload_url = "https://cloud-api.yandex.net/v1/disk/resources/upload"
        headers = self.get_headers()
        params = {"path": disk_file_path, "overwrite": "true"}
        response = requests.get(upload_url, headers=headers, params=params)
        pprint(response.json())
        return response.json()

    def upload_file_to_disk(self, disk_file_path, filename):
        href_json = self._get_upload_link(disk_file_path=disk_file_path)
        href = href_json["href"]
        response = requests.put(href, data=open(filename, 'rb'))
        response.raise_for_status()
        if response.status_code == 201:
            print("Успех")


if __name__ == "__main__":
    ya = YandexDisk(token=TOKEN)
    ya.upload_file_to_disk('dik/test2.txt', 'test.txt')
```