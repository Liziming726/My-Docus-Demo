---
title: My First Project experience
date: 2022-10-10
tags: [文章分享]
cover: ""
top_img: false
categories: [文章分享]
---

>This is my first participation in the project development record.   I'll record some of the gains here

---

## 1. The environment

- Development environment
- Test environment
- production environment

Development environment -> Test environment -> Production environment

> ⌨ Development environment is the development environment of the project, which is used to develop the project. The configuration can be relatively arbitrary. For the convenience of development and debugging, all error reporting and testing tools are generally opened, Which is the most basic enviroment.

> 🛠 Test environment is the test environment of the project, which is used to test the project. The environment is clone a production configuration and an application doesn't work well in the test environment,you can't release it to the production server. It's a transition from development to production.

> 🎉 Production environment is the production environment of the project, which is used to develop the project.  The production environment is the most important environment that provides external services . Generally , the error report is disabled and the error log is enabled . The deployment branch is usually the master branch.

As you can see , the project is in production

## 2. About nodejs error reporting issues


`Error: PostCSS plugin remove-google-fonts requires PostCSS 8.`
![111](https://s1.imgbed.xyz/2022/10/11/AzxIN.jpg)

```js
The solution is to degrade the postcss-remove-google-fonts version to ^1.1.4
```
---

## 😋 Hey , I'm come back ! Today is October 11 .

>I finished the first module of the project , this process is really interesting and has a risk of failure , but I have to say that I have learned a lot of things , I will continue to work hard , I hope I can do better in the future.  Then , Let's talk about the technology stack of the project .

- The API is so amazing , I can't wait to use it. The code in this column I end 

```js
return new Promise((resolve, reject) => {
    request
      .post(apis.getscholarismInfoType, obj)
      .then((res) => {
        resolve(res);
      })
      .catch((err) => {
        reject(err);
      });
  });
```

All the Data Structures are following the API , we must follow the API to develop the project , otherwise it will be a mess.

---
- The second stone is file upload . Let's take a look at the code 

```js
function beforeUpload(file: any) {
    const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png';
    if (!isJpgOrPng) {
      message.error('You can only upload JPG/PNG file!');
    }
    const isLt2M = file.size / 1024 / 1024 < 5;
    if (!isLt2M) {
      message.error('Image must smaller than 5MB!');
    }
    return isJpgOrPng && isLt2M;
  }
  function getBase64(img: any, callback: any) {
    const reader = new FileReader();
    reader.addEventListener('load', () => callback(reader.result));
    reader.readAsDataURL(img);
  }
  const handleChange = (info: any) => {
    if (info.file.status === 'uploading') {
      setUpLoading(true);
      return;
    }
    if (info.file.status === 'done') {
      // Get this url from response in real world.
      getBase64(info.file.originFileObj, (imageUrl: string) => {
        setUpLoading(false);
        setImageUrl(imageUrl);
      });
    }
    if (info.file.status === 'error') {
      setUpLoading(false);
      notification.error({
        message: 'Server Error',
        description: info.file.response.message,
      });
      return;
    }
  };
```

Ok , I'll go for a period of time , I'll come back later . 🎠