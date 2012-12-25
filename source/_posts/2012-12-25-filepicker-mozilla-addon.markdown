---
layout: post
title: "filepicker for mozilla addon"
date: 2012-12-25 23:06
comments: true
categories: [mozilla,addon]
tags: [mozilla,addon]
---
If your develpoing a mozilla addon, and want a file picker you can use the filePicker module. It uses component interface. You can find more info 
[nsIFilePicker](https://developer.mozilla.org/en-US/docs/XPCOM_Interface_Reference/nsIFilePicker).


{% codeblock filePicker lang:js https://github.com/padwan-ragavan/omnisequences/blob/master/lib/filePicker.js filePicker.js %}
const {Cc,Ci} = require("chrome");
const windowUtils = require("window-utils");

var getExportFileName = function() {
    var filePicker = Cc['@mozilla.org/filepicker;1']
        .createInstance(Ci.nsIFilePicker);
    filePicker.init(windowUtils.activeBrowserWindow, "Export Omnisequences", Ci.nsIFilePicker.modeSave);
    filePicker.appendFilter("*.json", "*.json");
    var show = filePicker.show();
    if(show != Ci.nsIFilePicker.returnCancel) {
        var fileName = filePicker.file.persistentDescriptor;
        if(/.json$/.test(fileName)) {
            return fileName;
        } else {
            return fileName + ".json";
        }
    }
    return null;
};

var getImportFileName = function() {
    var filePicker = Cc['@mozilla.org/filepicker;1']
        .createInstance(Ci.nsIFilePicker);
    filePicker.init(windowUtils.activeBrowserWindow, "Export Omnisequences", Ci.nsIFilePicker.modeOpen);
    filePicker.appendFilter("*.json", "*.json");
    var show = filePicker.show();
    if(show != Ci.nsIFilePicker.returnCancel) {
        return filePicker.file.persistentDescriptor;
    }
    return null;
};

exports.getExportFileName = getExportFileName;
exports.getImportFileName = getImportFileName;
{% endcodeblock %}