# Подключения AWS S3 к django проекту, необходимо

1. Заходим на <https://aws.amazon.com/>
2. Настройка/создание S3 хранилища

   1. Переходим к S3(можно через поиск).
   2. Создаем Корзину.

      1. Нажимаем на кнопку Create Bucket.
      2. Указываем необходимые данные.
      3. Создаем корзину.

3. Получаем доступ к AWS S3 корзине через AIM.

   1. Переходим к AIM(можно через поиск).
   2. Переходим к пользователям в боковом меню.
   3. Добавляем нового пользователя.

      1. Нажимам на кнопку Add User.
      2. Вводим нужные данные.
      3. В Access type указываем Programmatic access.
      4. В разделе *Установить разрешения* выберите *Прикрепить существующие политики напрямую*.
      5. Установите флажок AWSS3FullAcess.
      6. Создаем пользователя.
      7. Сохраняем access key и secret key куда-нибудь.

4. Настройка django

   1. Установите django-storerages: `pip install django-storages`

   2. В settings.py добавьте:

      `AWS_ACCESS_KEY_ID = <YOUR AWS ACCESS KEY>`

      `AWS_SECRET_ACCESS_KEY = <YOUR AWS SECRET KEY>`

      `AWS_STORAGE_BUCKET_NAME = <YOUR AWS S3 BUCKET NAME>`

      `AWS_S3_SIGNATURE_VERSION = 's3v4'`

      `AWS_S3_REGION_NAME = <YOUR AWS S3 BUCKET LOCATION>`

      `AWS_S3_FILE_OVERWRITE = False`

      `AWS_DEFAULT_ACL = None`

      `AWS_S3_VERIFY = True`
  
      `DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'`

