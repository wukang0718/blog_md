```javascript
	function myFileInput(obj) {
    if (obj.initialData) {
        initialPreview = getInitialPreview(obj.initialData);
    }
    initFileInput(obj);
    $('.kv-file-download.btn.btn-sm.btn-kv.btn-default.btn-outline-secondary').attr('target', '_self');
    $('.file-caption-name').attr('readonly', 'readonly');
}

function getInitialPreview(arr) {
    var array = [];
    for (var i = 0; i < arr.length; i++) {
        array.push('<img src="" />');
    }
    return array
}

var uploadObj = {
    docId: null,
    userKey: userData.userKey
};

var initialPreview = [];

function initFileInput(obj) {
    var control = $('#' + obj.id);
    var options = Object.assign({
        language: 'zh', //设置语言
        uploadUrl: 'localhost', //上传的地址
        allowedFileExtensions: ['jpg', 'gif', 'png', 'doc', 'docx', 'pdf', 'ppt', 'pptx', 'txt', 'exe', 'xls', 'xlsx'],//接收的文件后缀
        maxFilesNum: 5,//上传最大的文件数量
        uploadAsync: true, //默认异步上传
        showUpload: true, //是否显示上传按钮
        showRemove: true, //显示移除按钮
        showPreview: true, //是否显示预览
        showCaption: false,//是否显示标题
        showBrowse: true, // 是否显示浏览按钮
        showCancel: false, // 是否显示取消按钮
        showClose: false, // 是否显示关闭按钮
        initialPreviewAsData: false,
        initialPreview: initialPreview, // 回显的数据的内容
        // initialPreviewDownloadUrl: '', // 文件下载的路径
        initialPreviewConfig: obj.initialData,  // 数据回显显示的配置
        /*[ // downloadUrl文件下载的路径
         {type: "text", caption: "Image-1.jpg", downloadUrl:downloadUrl('fileDownload'), key: 1, code: 1, extra: {id: 1}},
         // {type: "text", caption: "Image-2.jpg", size: 817000, key: '1519713821098wwp4h8v', code: 1, extra: {id: 1}},  // set as raw markup
         // {type: "text", size: 1430, caption: "LoremIpsum.txt", key: 3, code: 1, extra: {id: 1}},
         // {type: "text", size: 102400, caption: "123.docx", key: '1519788281200pwxfx87', code: 1, extra: {id: 1}}
         ],*/
        browseClass: "btn btn-mybtn", //按钮样式
        //dropZoneEnabled: true,//是否显示拖拽区域
        //minImageWidth: 50, //图片的最小宽度
        //minImageHeight: 50,//图片的最小高度
        //maxImageWidth: 1000,//图片的最大宽度
        //maxImageHeight: 1000,//图片的最大高度
        maxFileSize: 0,//单位为kb，如果为0表示不限制文件大小
        //minFileCount: 0,
        maxFileCount: 10, //表示允许同时上传的最大文件个数
        enctype: 'multipart/form-data',
        uploadExtraData: {}, // 文件上传携带的其他参数
        validateInitialCount: true,
        previewFileIcon: "<i class='glyphicon glyphicon-king'></i>",
        msgFilesTooMany: "选择上传的文件数量({n}) 超过允许的最大数值{m}！",
        deleteUrl: urlRandom('SendFile/deleteSendDocAttachment'), // 文件删除的路径
        ajaxDeleteSettings: {
            type: 'POST',
            contentType: 'application/json;charset=utf-8',
            beforeSend: function (xhr, settings,) {
                var data = settings.data;
                var dataArr = data.split("&");
                var dataObj = {};
                for (var i = 0; i < dataArr.length; i++) {
                    var arr = dataArr[i].split("=");
                    dataObj[arr[0]] = arr[1];
                }
                settings.data = JSON.stringify(dataObj);
            },
            success: function (data) {
                if (data.code === '00000000') {
                    alert("删除成功");
                }
            }
        },
        mergeAjaxDeleteCallbacks: 'before'

    }, obj);
    control.fileinput(options).on('filepreupload', function (event, data, previewId, index) {     //上传中
        var form = data.form, files = data.files, extra = data.extra,
            response = data.response, reader = data.reader;
        console.log('文件正在上传');
    }).on("fileuploaded", function (event, data, previewId, index) {    //一个文件上传成功
        console.log('文件上传成功！', data, previewId, index);
        //  假设后台返回上传失败
        // retryUpload(index)
    }).on('fileerror', function (event, data, msg) {  //一个文件上传失败
        console.log('文件上传失败！', event, data, msg);
        $.ajax({
            type: 'POST',
            data: data,
            contentType: 'multipart/form-data',
            dataType: 'JSON',
            success: function () {
                console.log('success');
            },
            error: function () {
                console.log('error');
            }
        })
    }).on("filebatchselected", function (event, files) { // 方法在选择文件后触发
        // console.log(event, files);
    }).on('filebatchpreupload', function(event, data) { //该方法将在上传之前触发
        if (!data.files.length) {
            return {
                msg: "无附件",
                data: {}
            }
        }
    })
}

function retryUpload(index) {
    $('.file-drop-zone').find('.kv-file-upload.btn.btn-sm.btn-kv.btn-default.btn-outline-secondary').eq(index * 2).click();
}

function downloadUrl(url) {
    return url + '?userKey=' + userData.userKey
}

```