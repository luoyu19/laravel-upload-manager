# laravel-upload-manager
通过API对文件进行“上传、验证、储存、管理”操作。
Upload, validate, storage, manage by API for Laravel 5.1

## 需求 Requirement

1. Laravel 5.1

## 安装 Install

1. composer require zgldh/laravel-upload-manager
2. ```app.php```  ```'providers' => [ 'zgldh\UploadManager\UploadManagerServiceProvider']```
3. php artisan vendor:publish --provider="zgldh\UploadManager\UploadManagerServiceProvider"
4. php artisan migrate
5. Done

## 用法 Usage

1. 上传一个文件 Upload and store a file.
    
    ```php
     
        use zgldh\UploadManager\UploadManager;
        
        class UploadController extend Controller
        {
            public function postUpload(Request $request)
            {
                $file = $request->file('avatar');
                $uploadManager = UploadManager::getInstance();
                $upload = $uploadManager->upload($file);
                $upload->save();
                return $upload;
            }
        }
    ```
 
2. 从一个URL获取并保存文件 Fetch and store a file from a URL
    
    ```php
     
        use zgldh\UploadManager\UploadManager;
        
        class UploadController extend Controller
        {
            public function postUpload(Request $request)
            {
                $fileUrl = $request->input('url');
                $uploadManager = UploadManager::getInstance();
                $upload = $uploadManager->upload($fileUrl);
                $upload->save();
                return $upload;
            }
        }
    ```
 
3. 更新一个上传对象 Update a upload object
    
    ```php
     
        use App\Upload;
        use zgldh\UploadManager\UploadManager;
        
        class UploadController extend Controller
        {
            public function postUpload(Request $request)
            {
                $uploadId = $request->input('id');
                $file = $request->file('avatar');
                
                $uploadManager = UploadManager::getInstance();
                $upload = Upload::find($uploadId);
                if($uploadManager->upload($upload, $file))
                {
                    $upload->save();
                    return $upload;
                }
                return ['result'=>false];
            }
        }
    ```
 
4. 用从一个URL获取到的文件来更新一个上传对象 Update a upload object from a URL
    
    ```php
     
        use App\Upload;
        use zgldh\UploadManager\UploadManager;
        
        class UploadController extend Controller
        {
            public function postUpload(Request $request)
            {
                $uploadId = $request->input('id');
                $fileUrl = $request->input('url');
                
                $uploadManager = UploadManager::getInstance();
                $upload = Upload::find($uploadId);
                if($uploadManager->upload($upload, $fileUrl))
                {
                    $upload->save();
                    return $upload;
                }
                return ['result'=>false];
            }
        }
    ```
    
5. 数据验证 Validation
    
    ```php
    
        use zgldh\UploadManager\UploadManager;
        
        class UploadController extend Controller
        {
            public function postUpload(Request $request)
            {
                $file = $request->file('avatar');
                $uploadManager = UploadManager::getInstance();
                $upload = $uploadManager->withValidator('image')->upload($file);    //加上验证组
                
                if($upload)
                {
                    $upload->save();
                    return $upload;
                }
                else
                {
                    $errorMessages = $uploadManager->getErrors();                   //得到所有错误信息
                    $errorMessage = $uploadManager->getFirstErrorMessage();         //得到第一条错误信息
                    throw new \Exception($errorMessage);
                }
            }
        }
    ```
    
## 配置 Configuration

1. ``` config/upload.php ```

    请查看源文件注释

2. ``` App\Upload ```

    可以在里面写自己喜欢的函数
    
3. ``` UploadStrategy.php ```

    通常需要你亲自扩展一个出来。如：
    
    ```php
        
        <?php namespace App\Extensions;
        
        use zgldh\UploadManager\UploadStrategy as BaseUploadStrategy;
        use zgldh\UploadManager\UploadStrategyInterface;
        
        class UploadStrategy extends BaseUploadStrategy implements UploadStrategyInterface
        {
        
            /**
             * 生成储存的相对路径
             * @param $filename
             * @return string
             */
            public function makeStorePath($filename)
            {
                $path = 'i/' . $filename;
                return $path;
            }
        
            /**
             * 得到 disk localuploads 内上传的文件的URL
             * @param $path
             * @return string
             */
            public function getLocaluploadsUrl($path)
            {
                $url = url('uploads/' . $path);
                return $url;
            }
        
            /**
             * 得到 disk qiniu 内上传的文件的URL
             * @param $path
             * @return string
             */
            public function getQiniuUrl($path)
            {
                $url = 'http://' . trim(\Config::get('filesystems.disks.qiniu.domain'), '/') . '/' . trim($path, '/');
                return $url;
            }
        } 
    ```
    
    然后在 ``` config/upload.php ``` 里面配置 ``` upload_strategy ``` 为你自己扩展的类即可。
    
    
待续

    