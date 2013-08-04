scalatra-forms
==============

A library to validate and map request parameters for Scalatra.

At first, add the following dependency into your build.sbt to use scalatra-forms
and put <a href="https://raw.github.com/takezoe/scalatra-forms/master/src/main/resources/validation.js">validation.js</a> into your project.

```scala
resolvers += "amateras-repo" at "http://amateras.sourceforge.jp/mvn/"

libraryDependencies += "jp.sf.amateras" %% "scalatra-forms" % "0.0.1"
```

Define a form mapping at first. It's a similar to Play2, but scalatra-forms is more flexible.

```scala
import jp.sf.amateras.scalatra.forms._

case class RegisterForm(name: String, description: String)

val form = mapping(
  "name"        -> text(required, maxlength(40)), 
  "description" -> text()
)(RegisterForm.apply)
```

Next, create a servlet (or filter) which extends ScalatraServlet (or ScalatraFilter).
It also mixed in ```jp.sf.amateras.scalatra.forms.FormSupport``` or ```jp.sf.amateras.scalatra.forms.ClientSideValidationFormSupport```.
The object which is mapped request parameters is passed as an argument of action.

```scala
class RegisterServlet extends ScalatraServlet with ClientSideValidationFormSupport {
  post("/register", form) { form: RegisterForm =>
    ...
  }
}
```

In the HTML, add ```validation="true"``` to your form.
scalatra-forms registers a submit event listener to validate form contents.
This listener posts all the form contents to ```FORM_ACTION/validate```.
This action is registered by scalatra-forms automatically to validate form contents.
It returns validation results as JSON.

In the client side, scalatra-forms puts error messages into ```span#error-FIELD_NAME```.

```scala
<form method="POST" action="/register" validation="true">
  Name: <input type="name" type="text">
  <span class="error" id="error-name"></span>
  <br/>
  Description: <input type="description" type="text">
  <span class="error" id="error-description"></span>
  <br/>
  <input type="submit" value="Register"/>
</form>
```

For the Ajax action, use ```ajaxGet``` or ```ajaxPost``` instead of ```get``` or ```post```.
Actions which defined by ```ajaxGet``` or ```ajaxPost``` return validation result as JSON response.

```scala
class RegisterServlet extends ScalatraServlet with ClientSideValidationFormSupport {
  ajaxPost("/register", form) { form: RegisterForm =>
    ...
  }
}
```

In the client side, you can render error messages using ```displayErrors()```. 

```js
$('#register').click(function(e){
  $.ajax($(this).attr('action'), {
    type: 'POST',
    data: {
      name       : $('#name').val(),
      description: $('#description').val()
    }
  })
  .done(function(data){
    $('#result').text('Registered!');
  })
  .fail(function(data, status){
    displayErrors($.parseJSON(data.responseText));
  });
});
```

Release Notes
--------

# 0.0.1 - 04 Aug 2013

* This is the first public release.
