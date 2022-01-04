# Настрока FastAPI приложения.

1. Настроить окружения:
   1. Установить pipenv менеджер пакетов: `pip install pipenv`
   2. Создать виртуальное окружение: `pipenv install`
   3. Создать директории для дальнейшего использования: `mkdir models && mkdir crud && mkdir settings && mkdir routes`
      - models: Содержаться модели каждого из модулей.
      - crud: Содержаться методы работы с моделями модулей.
      - settings: Содержаться все конфиги.
   4. Создать разные конфиги настроек: `touch common.py && touch local.py && touch stage.py && touch test.py && touch prod.py`
   
2. Установить FastAPI: `pipenv install fastapi uvicorn`

3. Настроить главный main.py:
   1. Создать файл main.py: `touch main.py`
   2. Инициализировать fast api в main.py: 
    ```python
    from fastapi import FastAPI
    from starlette.middleware.cors import CORSMiddleware
    
    app = FastAPI()
    
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
    ```
4. Добавить файл с часто используемыми командами:
   1. Создать файл: `touch manage.py`
   2. Вставить следующий код в файл manage.py:
    ```python
    import argparse
    import os
    
    
    class Commands:
        @staticmethod
        def run(args: argparse.Namespace):
            port = args.port
            host = args.host
            os.system(f'sudo uvicorn main:app --reload --workers 1 --host {host} --port {port}')
    
    
    def _init_parser():
        parser = argparse.ArgumentParser(prog='FastAPIChat', description='App management')
    
        parser.add_argument('command', help='Commands to execute: run, migrate, make_migrations')
    
        # run command args
        parser.add_argument('--port', action='store', type=int, default=80, help='Server post')
        parser.add_argument('--host', action='store', type=str, default='0.0.0.0', help='Server host')
    
        return parser
    
    
    def main():
        parser = _init_parser()
        args = parser.parse_args()
    
        getattr(Commands, args.command)(args)
    
    
    if __name__ == "__main__":
        main()
    ```