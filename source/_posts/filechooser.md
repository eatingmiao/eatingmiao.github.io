title: Ionic HybridApp 选择与打开文件
tags: [AngularJs,Ionic,Cordova]
date: 2016-08-02
---
做了个手机端云盘，在选择和打开文件这里，又是个生僻的功能点……
寻觅了众多插件，原本用的这个filechooser：
* [Cordova Filechooser Plugin](https://github.com/don/cordova-filechooser)

界面直接调用安卓手机系统的文件管理器，代码也十分简单，但经测试后发现，多媒体文件没取到实际路径和名称，这主要是因为安卓多媒体文件有特定的URI，以content开头（比如安卓4.4以下的系统版本的图片URI可以为content://media/extenral/images/media/18726，但4.4以后的版本又有所不同），将会交给安卓的内容提供器（如MediaStore）使用。这种多媒体名称格式也没法从中知道具体的文件类型，这样的话，在做打开文件的功能点时，就缺少一个文件类型的参数。
所以换成了这个文件选择插件：
* [MaginSoft MFilechooser Plugin](https://github.com/MaginSoft/MFileChooser)

这个插件自带了一个界面，以后可以根据自己APP的需要改下UI风格，路径和名称都能按实际返回。
选择文件后，用cordovaFileTransfer上传到服务端，查看文件时，第一次打开需要下载到本地，然后用cordovaFileOpener2选择程序打开相应的文件，具体代码如下：

	// 文件需下载到本地打开
	document.addEventListener("deviceready", function () {
	// 保存路径
	var path = cordova.file.externalApplicationStorageDirectory; 
	// 打开文件
	function openFile() {
		var MIMEtype = $scope.getFileType(item.fileName.substr(item.fileName.lastIndexOf('.')+1));
		if (MIMEtype) {
			$cordovaFileOpener2.open(
				path + item.fileName,
				MIMEtype
			).then(true, function(err) {
				toastr.error(err.message);
			});
		} else {
			toastr.error("无法匹配打开此文件的本地程序");
		}
	};
	// 检查本地是否已下载文件
	$cordovaFile.checkFile(path, item.fileName)
		.then(function (success) {
			openFile();
		}, function (error) {
			// 下载地址
			var uri = Constants.server + item.fileUrl;
			// 下载文件
			$cordovaFileTransfer.download(uri, path + item.fileName, {}, true)
				.then(function (result) {
					toastr.success("下载成功");
					openFile();
				}, function (err) {
					toastr.error("下载失败，请重试");
			});
		});
	}, false);

这里原本想用FileSystem API来进行文件的缓存的，结果打开文件时会返回NOT FOUND，查看了文件夹的确是下载成功了，想想应该是因为没有权限读取吧。
