# Class for Laravel that allows you easily uploads files via ajax.
https://blueimp.github.io/jQuery-File-Upload/
File Upload widget with multiple file selection, drag&drop support, progress bars, validation and preview images, audio and video for jQuery.
Supports cross-domain, chunked and resumable file uploads and client-side image resizing.

## -- Short installation instruction

1. Download and place UploadHandler.php to app/Classes/UploadHandler.php
2. Extend it in your controller (class YourController extends UploadHandler)

### - Usage example

```php
<?php

namespace App\Http\Controllers;

use App\Classes\UploadHandler;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Illuminate\Support\Env;
use Intervention\Image\Facades\Image;

class YourController extends UploadHandler
{

    public function upload(Request $request)
    {
            if ($request->hasFile('files')) {

            // seting options for your upload, where to place uploaded images (dir), options can be extended via UploadHabdler class
            $this->setoptions = array(
                'script_url' => Env::get('APP_URL') . '/',
                'upload_dir' => dirname($this->get_server_var('SCRIPT_FILENAME')) . '/images/big-photos/',
                'upload_url' => '/big-photos/',
            );
            
            // preparing files to be uploaded
            $upload = $this->get_upload_data('files');
            
            // Parse the Content-Disposition header, if available:
            $content_disposition_header = $this->get_server_var('HTTP_CONTENT_DISPOSITION');
            $file_name = $content_disposition_header ?
                rawurldecode(preg_replace(
                    '/(^[^"]+")|("$)/',
                    '',
                    $content_disposition_header
                )) : null;
            
            // Parse the Content-Range header, which has the following form:
            // Content-Range: bytes 0-524287/2000000
            $content_range_header = $this->get_server_var('HTTP_CONTENT_RANGE');
            $content_range = $content_range_header ?
                preg_split('/[^0-9]+/', $content_range_header) : null;
            $size = @$content_range[3];
            $files = [];
            
            if ($upload) {
                if (is_array($upload['tmp_name'])) {
                    // param_name is an array identifier like "files[]",
                    // $upload is a multi-dimensional array:
                    foreach ($upload['tmp_name'] as $index => $value) {
                        $files[] = $this->handle_file_upload(
                            $upload['tmp_name'][$index],
                            $file_name ? $file_name : $upload['name'][$index],
                            $size ? $size : $upload['size'][$index],
                            $upload['type'][$index],
                            $upload['error'][$index],
                            $index,
                            $content_range
                        );
                    }
                } else {
                    // param_name is a single object identifier like "file",
                    // $upload is a one-dimensional array:
                    $files[] = $this->handle_file_upload(
                        isset($upload['tmp_name']) ? $upload['tmp_name'] : null,
                        $file_name ? $file_name : (isset($upload['name']) ?
                            $upload['name'] : null),
                        $size ? $size : (isset($upload['size']) ?
                            $upload['size'] : $this->get_server_var('CONTENT_LENGTH')),
                        isset($upload['type']) ?
                            $upload['type'] : $this->get_server_var('CONTENT_TYPE'),
                        isset($upload['error']) ? $upload['error'] : null,
                        null,
                        $content_range
                    );
                }

                foreach ($files as $file) {
                    // if you want watermark photos
                    if ($file->print_logo) {
                        $img = Image::make($this->setoptions['upload_dir'] . $file->name);
                        /* watermark */
                        $img->insert('path to your logo watermark'), 'bottom-center', 20, 20);
                        $img->save($this->setoptions['upload_dir'] . $file->name);
                    }

                    // your model code - inserting images to database
                }

            }
            
            // returning response  to view it in template
            $response = array($this->options['param_name'] => $files);
            
            return $response;
        }
        
     }
        
}
