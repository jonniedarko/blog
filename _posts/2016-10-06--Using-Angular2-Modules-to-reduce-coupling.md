---
layout: post
title:  "Managing Angular2 dependiences with Modules"
date:   2016-10-01 12:18:15
categories: angular2 material
---


### Update Nov 2016 ##
While this approach can still be useful for other modules or dependiences it is no longer required for Angular2 Material, now we can just use the `MaterialModule.forRoot()` function in out imports.
```
...
//import { MaterialModule} from '../material'; // this can be replaced with
import { MaterialModule} from '@angular/material';
...

@NgModule({
  imports:[
    FormsModule,
    ReactiveFormsModule,
    BrowserModule,
    MaterialModule.forRoot(),
    routing
  ],
  declarations: VIEW_DECLARATIONS,
  providers: VIEW_PROVIDERS
})

```
** end update **

So I recently jumped head 1st into angular 2 and have really tried to embrace the recent modularization push with `@NgModule` and have been trying to reduce the dependencies within my app while trying to keep things as reuseable and testable as possible. With that in mind and the fact that I was starting to find my AppComponent was getting very large and distracting to look at with all my different imports I decided to break up my componets into different modules. It started with `@angular2-material`. They have conveniently packaged each of the Material design components into their own modules but I took it a step further by wrapping this into a `Material-Module`.

```js
import { NgModule } from '@angular/core';

// Material-Componet Modules
import { MdCardModule } from '@angular2-material/card';
import { MdButtonModule } from '@angular2-material/button';
import { MdInputModule } from '@angular2-material/input';
import { MdToolbarModule } from '@angular2-material/toolbar';
import { MdListModule } from '@angular2-material/list';
import { MdIconModule, MdIconRegistry } from '@angular2-material/icon';



const MATERIAL_UI_MODULES = [
  MdCardModule,
  MdButtonModule,
  MdInputModule,
  MdToolbarModule,
  MdIconModule,
  MdListModule
];

const MATERIAL_UI_REGISTRIES = [
  MdIconRegistry
];

@NgModule({
  imports: [
    ...MATERIAL_UI_MODULES,
  ],
  providers: MATERIAL_UI_REGISTRIES,
  exports: [
    ...MATERIAL_UI_MODULES,
  ]
})
export class MaterialModule {

}
```

And now in my I simple need to import on single module into my `app.module`. But I thought, why stop there and so I also did this with my own "pure" components and my views. The views Module is particularly interesting as I was able to put the routing here too. Check it out below:

```js
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { HomeComponent} from './+home';
import {LoginComponent} from './+login';
import {ProfileComponent} from './+profile';
import {SignUpComponent} from './+signup';

import { Routes, RouterModule } from '@angular/router';
import { AUTH_PROVIDERS } from 'angular2-jwt';
import { AuthGuard } from '../common/auth.guard';
import { AuthActions } from '../models/current-user';
import { AuthService } from '../services/auth.service';
import { MaterialModule} from '../material';
import { SafeJsonPipe, PrettyJsonPipe, PrettyJsonComponent } from '../pipes';

export const appRoutes: Routes = [
  { path: '', component:  HomeComponent },
  { path: 'login',  component: LoginComponent },
  { path: 'signup',  component: SignUpComponent },
  { path: 'profile',   component: ProfileComponent, canActivate: [AuthGuard]},
  { path: '**',     component: LoginComponent },
];

export const appRoutingProviders: any[] = [

];

export const routing = RouterModule.forRoot(appRoutes);

export const VIEW_DECLARATIONS = [
  HomeComponent, LoginComponent, ProfileComponent, SignUpComponent,
  PrettyJsonComponent,
  SafeJsonPipe, PrettyJsonPipe
];

export const VIEW_PROVIDERS = [
  appRoutingProviders,
  AUTH_PROVIDERS,
  AuthService,
  AuthActions,
  AuthGuard
];

@NgModule({
  imports:[
    FormsModule,
    ReactiveFormsModule,
    BrowserModule,
    MaterialModule,
    routing
  ],
  declarations: VIEW_DECLARATIONS,
  providers: VIEW_PROVIDERS
})
export class ViewsModule {

}
```

Now this is still a work in progress, as you can see from my views module I have more possible module abstractions, for example the pipes and services. But it does make for less "distraction" in my componets which I am definitly enjoying.

