.. title:: Filestorage - Storage system

.. meta::
    :description: Filestorage - storage driver and storage location - dframeframework.com
    :keywords: dframe, filestorage, configuration, php, php7, local storage, stylist image, images, uploads 

A library used for supporting files of any kind. The system is based on the basic method of file upload and reading. A few additional methods make this method convenient in use, ex. a mechanism that prevents you from accidentally overwriting a file. System will inform you that there's already a file like that in a given location. 
The way of storing image information can be any. It can be the usual file_exist or mysql.  An additional feature of the library is the fact that you can save the files in anyway (ftp, local, NullAdapter) through league/fliesystem, or create your own adapters.

Instalation
----------

From console level, launch the command composer* 

.. code-block:: bash

 $ composer require dframe/filestorage

Or download manual https://github.com/dframe/filestorage/releases

Dframe/fileStorage is an addition to the system mentioned above, thanks to which it's ready to use in any system after setting the config and the driver.

Configuration
----------

.. code-block:: php

 use League\Flysystem\Adapter\Local;
 use League\Flysystem\Cached\CachedAdapter;
 use League\Flysystem\Cached\Storage\Memory as CacheStore;
 use League\Flysystem\Filesystem;
 use Model\DatabaseDriver;

 // Create the original file store
 $localAdapter = new Local(__DIR__ . '/uploads');
 
 // Create the cache store
 $cacheStore = new CacheStore();
 
 // Decorate the adapter
 $cacheAdapter = new CachedAdapter(new Local(__DIR__ . '/public_html/cache'), $cacheStore);
 

 $config = [
     'pluginsDir' => __DIR__ . '/plugins',
     'adapters' => [
         'local' => new Filesystem($localAdapter),
         'cache' => new Filesystem($cacheAdapter), 
     ],
     'cache' => [
         'life' => 600 // in seconds
     ],
     'publicUrls' => [
         'local' => ''
     ]
 ];

 $FileStorage = new \Dframe\FileStorage\Storage(new DatabaseDriver, $config);
 $FileStorage->settings([
    'stylists' => [
        'simple' => \Dframe\FileStorage\Stylist\SimpleStylist::class
    ]
 ]);
     

Driver here you have example driver https://github.com/dframe/fileStorage/blob/master/examples/example1/app/Model/FileStorage/Drivers/DatabaseDriver.php 

And here is empty driver

.. code-block:: php
 
 namespace Model; 
 
 use Dframe\FileStorage\Drivers\DatabaseDriverInterface;
 
 class DatabaseDriver implements DatabaseDriverInterface
 {
     /**
      * @param      $adapter
      * @param      $path
      * @param bool $cache
      *
      * @return mixed
      */
     public function get($adapter, $path, $cache = false)
     {
         // TODO: Implement get() method.
     }
     
     /**
      * @param $adapter
      * @param $path
      * @param $mine
      * @param $stream
      *
      * @return mixed
      */
     public function put($adapter, $path, $mine, $stream)
     {
         // TODO: Implement put() method.
     }
     
     /**
      * @param $adapter
      * @param $originalId
      * @param $path
      * @param $mine
      * @param $stream
      *
      * @return mixed
      */
     public function cache($adapter, $originalId, $path, $mine, $stream)
     {
         // TODO: Implement cache() method.
     }
     
     /**
      * @param $adapter
      * @param $path
      *
      * @return mixed
      */
     public function drop($adapter, $path)
     {
         // TODO: Implement drop() method.
     }
     
 
Upload
----------

Putting a file in a local private catalogue, without access to http, a model used for that is available `here
<https://github.com/dframe/fileStorage/blob/master/examples/example1/app/Model/FileStorage/Drivers/DatabaseDriver.php>`_. The example below shows receiving an image from a php site through a form.

.. code-block:: php

 if (isset($_POST['upload'])) {
 
     if (!$FileStorage->isAllowedFileType($_FILES['file'], ['jpg' => ['image/jpeg', 'image/pjpeg']])) {
         exit(json_encode(['code' => 400, 'message' => 'Uploaded file is not a valid image. Only JPG files are allowed']));
     }
 
     $put = $FileStorage->put('local', $_FILES['file']['tmp_name'], 'images/' . $_FILES['file']['name']);
     if ($put['return'] == true) {
         exit(json_encode(['code' => 200, 'message' => 'File Uploaded']));
 
     } elseif ($put['return'] == false) {
 
         //I know file exist, try put forced
         $put = $FileStorage->put('local', $_FILES['file']['tmp_name'], 'images/' . $_FILES['file']['name'], true);
         if ($put['return'] == true) {
             exit(json_encode(['code' => 207, 'message' => 'File existed and was overwritten']));
         }
 
     }
 
     exit(json_encode(['code' => 500, 'message' => 'Internal Error']));
 }
 
Reading
----------

In order to read an image, we can do it in two ways. If the file was uploaded privately, without http access, we have to create controller that will download it and show it. For that, we have the code below.

.. code-block:: php

 exit($FileStorage->renderFile('images/path/name/screenshot.jpg', 'local'));
 
This code will return the original file to us, no matter if it's .jpg or .pdf

Image Processing
----------

The library has an additional feature of real-time image processing, thanks to the possibility of adding our own driver and ability to process our image in any way.

.. code-block:: php

 echo $FileStorage->image('images/path/name/screenshot.jpg')->stylist('square')->size('250x250')->display();
 
After processing, a link to a rendered image of 250x250 size will be returned.

Return array

.. code-block:: php

 echo $FileStorage->image('images/path/name/screenshot.jpg')->stylist('square')->size('250x250')->get();
 

