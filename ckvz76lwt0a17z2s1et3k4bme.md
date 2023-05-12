---
title: "How to download PDF files in Angular"
seoTitle: "How to download PDF files in Angular"
seoDescription: "How to download PDF files in Angular"
datePublished: Sun Nov 14 2021 12:11:56 GMT+0000 (Coordinated Universal Time)
cuid: ckvz76lwt0a17z2s1et3k4bme
slug: how-to-download-pdf-files-in-angular
tags: angular, pdf, files

---

**Use case:** *Parse the blob response or base64 encoded string received from backend and Download PDF file on the fly.*

The backend service sends blob data as response
> 
%PDF-1.5
%âãÏÓ
1 0 obj
/Type/Page/Parent 8 0 R /MediaBox[ 0 0 612 792]/Contents 9 0 R /Resources/XObject/img49352 7 0 R /img49350 6 0 R /img49348 5 0 R /img49347 3 0 R /Font/F2 2 0 R /F4 4 0 R 
endobj.................

or base64 encoded string
> 
data:application/pdf;base64,JVBERi0xLjUKJeLjz9MKMSAwI.....

First step here would be to process & parse response. If the response is base64 string, we first need to convert that to blob object as the file download is possible with blob data only!
```js
private processFileResponse(fileResponseData: any, fileName: string, render: string): void {
  if (render === 'base64') {
    this.base64Response = fileResponseData;
    const binaryString = window.atob(fileResponseData);
    const bytes = new Uint8Array(binaryString.length);
    const binaryToBlob = bytes.map((byte, i) => binaryString.charCodeAt(i));
    const blob = new Blob([binaryToBlob], { type: 'application/pdf' });
    this.downloadFile(blob, fileName, render);
  } else {
    const blob = new Blob([fileResponseData], { type: 'application/pdf' });
    this.downloadFile(blob, fileName, render);
  }
}
```

Once the blob object is ready, we can now either download the file or open the file in new tab and let user decide if they want to download that file. 

Lets go through scenario of downloading file directly:
1. prepare object URL using `createObjectURL()`
2. create an anchor tag and assign object URL to `href` attribute 
3. initiate the download 
```js
private downloadFile(blob: any, fileName: string): void {
  // IE Browser
  if (window.navigator && window.navigator.msSaveOrOpenBlob) {
    window.navigator.msSaveOrOpenBlob(blob, fileName);
    return;
  }
  // Other Browsers
  const url = (window.URL || window.webkitURL).createObjectURL(blob);
  const link = this.renderer.createElement('a');
  this.renderer.setAttribute(link, 'download', fileName);
  this.renderer.setAttribute(link, 'href', url);
  this.renderer.setAttribute(link, 'target', '_blank');
  this.renderer.appendChild(this.elementRef.nativeElement, link);
  link.click();
  this.renderer.removeChild(this.elementRef.nativeElement, link);

  setTimeout(() => {
    window.URL.revokeObjectURL(url);
  }, 1000);
}
```
> Once the process is completed, make sure you have revoked the object URL else that will pile up on the browser's memory 

If you don't intend to download the file and instead want the file to be open in new tab and let user decide to download that or simply read it and close the tab, below are the steps for this:
```js
private downloadFile(blob: any, fileName: string): void {
  // IE Browser
  if (window.navigator && window.navigator.msSaveOrOpenBlob) {
    window.navigator.msSaveOrOpenBlob(blob, fileName);
    return;
  }

  // Other Browsers
  const url = (window.URL || window.webkitURL).createObjectURL(blob);
  window.open(url, '_blank');

  // rewoke URL after 15 minutes
  setTimeout(() => {
    window.URL.revokeObjectURL(url);
  }, 15 * 60 * 1000);
}
```
I personally like this approach as it gives user more flexibility to decide if they are willing to download the file or just intend to read the content. Only downside to this approach is you cannot assign custom file name  to the downloaded file.

### Handling edge cases with IOS (apple) devices

Safari browser do not support the approach that we have implemented (using BLOB URLs) so far for security reasons. To enable downloading on Safari browsers, we make use of third-party library **file-saver-es**
```js
import { saveAs } from 'file-saver-es';
...
private downloadFile(blob: any, fileName: string): void {
  ...
  if (this.useFileSaverAPI()) {
    saveAs(blob, fileName);
  } else {
    // rest of devices and browsers
    window.open(url, '_blank');
  }
}
private useFileSaverAPI(): boolean {
  return (/(iPad|iPhone|iPod)/g.test(navigator.platform || navigator.userAgent) || ((navigator.platform || navigator.userAgent) === 'MacIntel' && navigator.maxTouchPoints > 1)) && !window.MSStream;
}
```

### Handling edge cases with old Android web-view (v5 and less)

For Android v5 and below, there are scenarios where web-view does not recognizes the blob object and hence the download fails. We need to convert that blob object to base64 string using FileReader api and then initiate the download.
```js
if (this.isAndroid()) {
  if (this.base64Response) {
    window.open(this.base64Response, '_blank');
  } else {
    const fileReader = new FileReader();
    fileReader.readAsDataURL(blob);
    fileReader.onloadend = () => {
      window.open(fileReader.result as any, '_blank');
    };
  }
} else {
  // rest of devices and browsers
  window.open(url, '_blank');
}

private isAndroid(): boolean {
  return (/(android)/i.test(navigator.platform || navigator.userAgent)
}
```

