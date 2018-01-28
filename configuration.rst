A library used for supporting files of any kind. The system is based on the basic method of file upload and reading. A few additional methods make this method convenient in use, ex. a mechanism that prevents you from accidentally overwriting a file. System will inform you that there's already a file like that in a given location. 
The way of storing image information can be any. It can be the usual file_exist or mysql.  An additional feature of the library is the fact that you can save the files in anyway (ftp, local, NullAdapter) through league/fliesystem, or create your own adapters.

Dframe/fileStorage is an addition to the system mentioned above, thanks to which it's ready to use in any system after setting the config and the driver.

Upload
^^^^^^^^^

Putting a file in a local private catalogue, without access to http, a model used for that is available here. The example below shows receiving an image from a php site through a form.

.. code-block:: php

 if(isset($_POST['upload'])){
    $fileStorage = new \Dframe\FileStorage\Storage($this->loadModel('FileStorage/Drivers/DatabaseDriver'));
    $put = $fileStorage->put('local', $_FILES['file']['tmp_name'], 'images/path/name.'.$extension);
    if($put['return'] == true)
       exit(json_encode(array('return' => '1', 'response' => 'File Upload OK')));
           
    exit(json_encode(array('return' => '0', 'response' => 'Error')));
 }
 
Reading
^^^^^^^^^^^^

In order to read an image, we can do it in two ways. If the file was uploaded privately, without http access, we have to create controller that will download it and show it. For that, we have the code below.

.. code-block:: php

 exit($fileStorage->renderFile('images/path/name/screenshot.jpg', 'local'));
This code will return the original file to us, no matter if it's .jpg or .pdf

Image Processing
^^^^^^^^^^^^^^

The library has an additional feature of real-time image processing, thanks to the possibility of adding our own driver and ability to process our image in any way.

.. code-block:: php

 echo $fileStorage->image('images/path/name/screenshot.jpg')->stylist('square')->size('250x250')->display();
After processing, a link to a rendered image of 250x250 size will be returned.
