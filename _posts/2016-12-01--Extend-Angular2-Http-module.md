---
layout: post
title:  "Extend Angular 2 Http"
date:   2016-10-01 12:18:15
categories: angular2 http headers
---
Often we need to set custom headers in our ajax requests and while in Angular2 we can pass these headers as options to 
every request we make it can get tedious and is even error prone if we approach it this way. There are many possible approahes 
for example one approach I have seen regularly is to implement a service that injects http and when you call `get`,  `post` or 
what ever other http method you need you wrap that from with in your custom service. Tedious and still Error prone as a 
developer may forget to import the custom http service or you may have to spend time reinventing existing functionality. If
only you could just extend the existing Http.

You can, using Typscript's `extends` we can just create a bare bone HttpClient that overrides the `request` function to attach
or extend the headers or what ever other options we want.

```ts
import {Injectable} from "@angular/core";
import {Http, Headers, RequestOptionsArgs, Request, Response, ConnectionBackend, RequestOptions} from "@angular/http";

import {Observable} from 'rxjs/Observable';

@Injectable()
export class HttpClient extends Http {

  constructor(protected _backend: ConnectionBackend, protected _defaultOptions: RequestOptions) {

    super(_backend, _defaultOptions);
  }

  _setCustomHeaders(options?: RequestOptionsArgs):RequestOptionsArgs{
    if(!options) {
      options = new RequestOptions({});
    }
    if(localStorage.getItem("id_token")) {

      if (!options.headers) {

        options.headers = new Headers();


      }
      options.headers.set("Authorization", localStorage.getItem("id_token"))
    }
    return options;
  }


  request(url: string|Request, options?: RequestOptionsArgs): Observable<Response> {
    options = this._setCustomHeaders(options);
    return super.request(url, options)
  }
}
```
In Angular the `http` provider is provided using a factory. So we will do the exact same the only difference is our factory will return a new HttpClient (which is an extended Http) instead of angulars Http.
Now in our Application's every time we inject Http, our custom HttpClient will be provided and noting special needs to be done

```ts
import { RequestOptions, Http, XHRBackend} from '@angular/http';
import {HttpClient} from './httpClient';
import { RequestOptions, Http, XHRBackend} from '@angular/http';
import {HttpClient} from './httpClient';//above snippet

function httpClientFactory(xhrBackend: XHRBackend, requestOptions: RequestOptions): Http {
  return new HttpClient(xhrBackend, requestOptions);
}

@NgModule({
  imports:[
    FormsModule,
    BrowserModule,
  ],
  declarations: APP_DECLARATIONS,
  bootstrap:[AppComponent],
  providers:[
     { provide: Http, useFactory: httpClientFactory, deps: [XHRBackend, RequestOptions]}
  ],
})
export class AppModule {
  constructor(){

  }
}
```


