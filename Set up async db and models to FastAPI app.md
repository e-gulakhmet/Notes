# Настройка асинхронной Postgres базы данных и моделей для нее в приложении FastAPI

1. Установить нужные пакеты: `pipenv install alembic sqlmodel asyncpg`
2. В файл settings/common.py, указать DATABASE_URL(пример: postgresql+asyncpg://main:main@localhost:5432/main)

3. Настроить подключение к базе данных:
   1. Создать файл db.py в проекте: `touch db.py`
   2. Вставить следующий код в файл db.py:
    ```python
    from sqlmodel import SQLModel

    from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
    from sqlalchemy.orm import sessionmaker
    
    from settings.core import settings
    
    engine = create_async_engine(settings.DATABASE_URL, echo=True, future=True)
    
    
    async def init_db():
        async with engine.begin() as conn:
            # await conn.run_sync(SQLModel.metadata.drop_all)
            await conn.run_sync(SQLModel.metadata.create_all)
    
    
    async def get_session() -> AsyncSession:
        async_session = sessionmaker(
            engine, class_=AsyncSession, expire_on_commit=False
        )
        async with async_session() as session:
            yield session
    ```

4. Настроить модели:
   Допустим нужно создать пользователя в базе данных и модели, которые будут работать с данными, которые приходят в json формате от фронта.
   1. Создать файл users.py в директории models: `touch users.py`
   2. Вставить следующее содержимое в файл users.py:
    ```python
    from sqlmodel import SQLModel, Field
    
    
    class BaseUser(SQLModel):  # Основная модель, от которой будут наследовать остальные модели бд и модели сериализации данных
        first_name: str
        last_name: str
        username: str = Field(nullable=False, min_length=3, max_length=100)
        email: str = Field(nullable=False, min_length=3, max_length=100)
    
    
    class User(BaseUser, table=True):  # Модель базы данных
        id: int = Field(primary_key=True)
        hashed_password: str = Field(nullable=False)
    
    
    class UserCreate(BaseUser):  # Модель сериализации данных для создания пользователя
        password: str
    
    
    class UserUpdate(BaseUser):  # Модель сериализации данных для обновления пользователя
        pass
    ```
   
5. Настроить интерфейс работы с данными в бд:
   1. В директории crud создать файл base.py: `touch base.py`
   2. Вставить следующий код в файл base.py:
   В данном файле находить основной класс, который представляет основные методы работы с данными в бд.
    ```python
    from typing import Any, Dict, Generic, List, Optional, Type, TypeVar, Union

    from fastapi.encoders import jsonable_encoder
    from sqlmodel import SQLModel
    from sqlmodel.ext.asyncio.session import AsyncSession
    
    ModelType = TypeVar("ModelType", bound=SQLModel)
    CreateSchemaType = TypeVar("CreateSchemaType", bound=SQLModel)
    UpdateSchemaType = TypeVar("UpdateSchemaType", bound=SQLModel)
    
    
    class CRUDBase(Generic[ModelType, CreateSchemaType, UpdateSchemaType]):
        def __init__(self, model: Type[ModelType]):
            """
            :param model: A SQLAlchemy model class
            """
    
            self.model = model
    
        async def get(self, db: AsyncSession, id: int) -> Optional[ModelType]:
            return db.query(self.model).filter(self.model.id == id).first()
    
        async def get_multi(self, db: AsyncSession, *, skip: int = 0, limit: int = 100) -> List[ModelType]:
            return db.query(self.model).offset(skip).limit(limit).all()
    
        async def create(self, db: AsyncSession, *, obj_in: CreateSchemaType) -> ModelType:
            obj_in_data = jsonable_encoder(obj_in)
            db_obj = self.model(**obj_in_data)  # type: ignore
            db.add(db_obj)
            await db.commit()
            await db.refresh(db_obj)
            return db_obj
    
        async def update(self, db: AsyncSession, *, db_obj: ModelType, obj_in: Union[UpdateSchemaType, Dict[str, Any]]
                         ) -> ModelType:
            obj_data = jsonable_encoder(db_obj)
            if isinstance(obj_in, dict):
                update_data = obj_in
            else:
                update_data = obj_in.dict(exclude_unset=True)
            for field in obj_data:
                if field in update_data:
                    setattr(db_obj, field, update_data[field])
            db.add(db_obj)
            await db.commit()
            await db.refresh(db_obj)
            return db_obj
    
        async def remove(self, db: AsyncSession, *, id: int) -> ModelType:
            obj = db.query(self.model).get(id)
            await db.delete(obj)
            await db.commit()
            return obj
    ```

   3. В директории crud создать файл users.py: `touch users.py`
   4. Вставить следующий код в файл base.py:
   В данном файле находить класс работы с данными пользователя в бд.
    ```python
    from crud.base import CRUDBase
    from models.users import User, UserCreate, UserUpdate
    
    
    class CRUDUser(CRUDBase[User, UserCreate, UserUpdate]):
        ...
    
    recipe = CRUDUser(User)
    ```   

6. Настроить миграции:
   1. Инициализировать миграции в проекте: `alembic init -t async migrations `
   2. Настроить конфиг миграций в файле alembic.ini:
     ```
    # A generic, single database configuration.
    
    [alembic]
    # path to migration scripts
    script_location = migrations
    
    # template used to generate migration files
    # file_template = %%(rev)s_%%(slug)s
    
    # sys.path path, will be prepended to sys.path if present.
    # defaults to the current working directory.
    prepend_sys_path = .
    
    # timezone to use when rendering the date within the migration file
    # as well as the filename.
    # If specified, requires the python-dateutil library that can be
    # installed by adding `alembic[tz]` to the pip requirements
    # string value is passed to dateutil.tz.gettz()
    # leave blank for localtime
    # timezone =
    
    # max length of characters to apply to the
    # "slug" field
    # truncate_slug_length = 40
    
    # set to 'true' to run the environment during
    # the 'revision' command, regardless of autogenerate
    # revision_environment = false
    
    # set to 'true' to allow .pyc and .pyo files without
    # a source .py file to be detected as revisions in the
    # versions/ directory
    # sourceless = false
    
    # version location specification; This defaults
    # to migrations/versions.  When using multiple version
    # directories, initial revisions must be specified with --version-path.
    # The path separator used here should be the separator specified by "version_path_separator"
    # version_locations = %(here)s/bar:%(here)s/bat:migrations/versions
    
    # version path separator; As mentioned above, this is the character used to split
    # version_locations. Valid values are:
    #
    # version_path_separator = :
    # version_path_separator = ;
    # version_path_separator = space
    version_path_separator = os  # default: use os.pathsep
    
    # the output encoding used when revision files
    # are written from script.py.mako
    # output_encoding = utf-8
    
    sqlalchemy.url = postgresql+asyncpg://main:main@localhost:5432/main
    
    
    [post_write_hooks]
    # post_write_hooks defines scripts or Python functions that are run
    # on newly generated revision scripts.  See the documentation for further
    # detail and examples
    
    # format using "black" - use the console_scripts runner, against the "black" entrypoint
    # hooks = black
    # black.type = console_scripts
    # black.entrypoint = black
    # black.options = -l 79 REVISION_SCRIPT_FILENAME
    
    # Logging configuration
    [loggers]
    keys = root,sqlalchemy,alembic
    
    [handlers]
    keys = console
    
    [formatters]
    keys = generic
    
    [logger_root]
    level = WARN
    handlers = console
    qualname =
    
    [logger_sqlalchemy]
    level = WARN
    handlers =
    qualname = sqlalchemy.engine
    
    [logger_alembic]
    level = INFO
    handlers =
    qualname = alembic
    
    [handler_console]
    class = StreamHandler
    args = (sys.stderr,)
    level = NOTSET
    formatter = generic
    
    [formatter_generic]
    format = %(levelname)-5.5s [%(name)s] %(message)s
    datefmt = %H:%M:%S
    ```
   3. Вставить слудующий код в файл migrations/env.py:
    ```python
    import asyncio
    from logging.config import fileConfig
    
    from sqlalchemy import engine_from_config
    from sqlalchemy import pool
    from sqlalchemy.ext.asyncio import AsyncEngine
    from sqlmodel import SQLModel
    
    from alembic import context
    
    # Add app db models here
    from models.users import User
    
    # this is the Alembic Config object, which provides
    # access to the values within the .ini file in use.
    config = context.config
    
    # Interpret the config file for Python logging.
    # This line sets up loggers basically.
    fileConfig(config.config_file_name)
    
    # add your model's MetaData object here
    # for 'autogenerate' support
    # from myapp import mymodel
    # target_metadata = mymodel.Base.metadata
    target_metadata = SQLModel.metadata
    
    # other values from the config, defined by the needs of env.py,
    # can be acquired:
    # my_important_option = config.get_main_option("my_important_option")
    # ... etc.
    
    
    def run_migrations_offline():
        """Run migrations in 'offline' mode.
    
        This configures the context with just a URL
        and not an Engine, though an Engine is acceptable
        here as well.  By skipping the Engine creation
        we don't even need a DBAPI to be available.
    
        Calls to context.execute() here emit the given string to the
        script output.
    
        """
        url = config.get_main_option("sqlalchemy.url")
        context.configure(
            url=url,
            target_metadata=target_metadata,
            literal_binds=True,
            dialect_opts={"paramstyle": "named"},
        )
    
        with context.begin_transaction():
            context.run_migrations()
    
    
    def do_run_migrations(connection):
        context.configure(connection=connection, target_metadata=target_metadata)
    
        with context.begin_transaction():
            context.run_migrations()
    
    
    async def run_migrations_online():
        """Run migrations in 'online' mode.
    
        In this scenario we need to create an Engine
        and associate a connection with the context.
    
        """
        connectable = AsyncEngine(
            engine_from_config(
                config.get_section(config.config_ini_section),
                prefix="sqlalchemy.",
                poolclass=pool.NullPool,
                future=True,
            )
        )
    
        async with connectable.connect() as connection:
            await connection.run_sync(do_run_migrations)
    
    
    if context.is_offline_mode():
        run_migrations_offline()
    else:
        asyncio.run(run_migrations_online())
    ```
   
    4. В файле migrations/script.py.mako, после `import sqlalchemy as sa` вставить `import sqlmodel`
    5. В файл manage.py в класс Commands, добавить следующий код:
    ```python
    @staticmethod
    def make_migrations(args: argparse.Namespace):
        message = args.message
        assert message, 'Set migration message arg(check help)'
        os.system(f'alembic revision --autogenerate -m "{message}"')

    @staticmethod
    def migrate():
        os.system('alembic upgrade head')
    ```
   
    5. В файл manage.py в класс методе init_parser, добавить следующий код:
    ```python
    # make_migrations command args
    parser.add_argument('--message', action='store', type=str, help='Migration message')
   ```