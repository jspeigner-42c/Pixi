# PIXI Intro Workshop	
#projects/pixi

## Database Vulnerabilities
1. Find the MongoDB Rest API
○	Using nmap find some open ports
```
		nmap pixi.ip -p1-65535
		27017/tcp open  mongod
		28017/tcp open  mongod
```
2. Browse to Mongo HTTP interface
○	http://localhost:28017/
○	http://localhost:28017/Pixidb/users/
■	What does this show us?

## Web App Vulnerabilities 
3. Find the Reflected XSS attack
○	Explain XSS
○	Explain difference between stored vs. reflected XSS
○	Search field is reflected, because when you do a normal search you see the query on the actual page, when the developer doesn't make sure all malicious input is properly escaped, it will be rendered by the browser, which is what executes the javascript. 
○	 `<script>alert('hi')</script>`

4. Find the Angular Constructor XSS attack
○	Angular uses constructors to instantiate and initialize fields in an angular template.  You can actually perform an XSS attack without using HTML but using angular constructor syntax.  
○	Angular templates can contain expressions - JavaScript-like code snippets inside double curly braces.  The text input `{{1+1}} `will be evaluated by the angular engine and result in 2
○	This means anyone able to inject double curly braces can execute Angular expressions. Angular expressions can't do much harm on their own, but when combined with a sandbox escape we can execute arbitrary JavaScript and do some serious damage.
○	`{{constructor.constructor('alert(1)')()}}`

5. Bypass the admin page restriction on the web app
○	Pixi's Express.js routes contains code preventing anyone from accessing /admin section unless they are actually administrators.  Trouble is that the code is just using regular expressions (RegEx) against the word 'admin'.
○	Therefore if you type in /ADMIN you should be able to bypass the unless routing control.

6. Find the actual administrator service account in a file
○	Pixi accidentally stores valuable keys and credentials in a publicly access web directory
○	http://localhost:8000/service.conf

7. Find the session secret for web app session cookies
○	Pixi accidentally stores valuable keys and credentials in a publicly access web directory
○	You get a hint that there are conf files by clicking on the secret menu item
○	http://localhost:8000/secret.conf

8. Using CSRF create a link to delete a photo you uploaded
○	Explain what CSRF is
○	When you are logged into your account and go to the profile page, you see a button on each photo to delete it. 
○	Right-clicking and inspecting that takes you to
○	`<a href="# “ ng-click="user_delete_photo(picture._id)" class="btn btn-info">Delete Photo <i class="fa fa-trash" aria-hidden="true"></i> </a>`
○	This means that angular calls a `user_delete_photo` method with a picture id parameter.
○	Digging further to find the `user_delete_photo` javascript
```
	$scope.user_delete_photo = function(photo){
	$http.get('/user_delete_photo/' + photo)
	  .then(function success(response){
		$scope.user_pictures();
		$scope.picture_alerts = [
	      { type: 'success',  msg: "Buh-Bye, Photo Deleted! " }
```
○	This shows that the GET method is used to delete the photo.  
○	Construct a URL with a valid photo id (that belongs to you) while you are logged in 
■	http://localhost:8000/user_delete_photo/photo_id
■	If you create these links and get people to click on them you have launched a CSRF attack.

9. Log in as another user by bypassing authentication
○	The login page is just a pure express.js form.  It will allow MongoDB injection for the same reason all injection happens, improper input sanitization.
○	By setting up a web proxy you see that `’user=email@address.com&pass=1234'` is in the POST body of the request.  
○	If you modify that to `user=email@address.com&pass[$ne]=`
Mongo uses `$ne` to specify not equal - `db.users.find({_id: {$ne: ''} })`

10. Upload something that is not a photo
○	Pixi allows for binary data upload.  Good practice would be to ensure that the file uploaded is actually what you'd expect (JPG/PNG), or do the conversion on the back-end.  Pixi allows arbitrary file upload, including executables.

11. Inject on the admin page to find all users
○	`http://localhost:8000/ADMIN/search?search[$ne]=1`

12. Inject on the admin page to find all likes
○	`http://localhost:8000/ADMIN/likes/search?search[$ne]=`

13. Inject on the admin page to find all loves
○	`http://localhost:8000/ADMIN/loves/search?search[$ne]=`

API Vulnerabilities
14. Make yourself an admin using by mass assigning via the API
○	Authenticate to the API using valid credentials
○	GET request to `localhost:8090/api/user/info`
○	Look at the is_admin field.
○	PUT request to `localhost:8090/api/user/edit_info`
○	Set is_admin to true.
○	Run the GET request again

15. Delete another Users photos
○	By looking at the API Definition you see that to delete photos you can use the DELETE Method: /api/picture/delete
○	If we try to leverage that method using
○	`localhost:8090/api/picture/delete?picture_id=11`
○	We see that because of underprotected API Routes, the picture is allowed to be deleted.  
16. Change another users profile
○	I forgot this… 
17. Inject on the admin api to find all users
○	`http://localhost:8000/api/ADMIN/search?search[$ne]=`
18. Inject on the admin api to find all likes
○	`http://localhost:8090/api/ADMIN/likes/search?search[$ne]=1`
19. Inject on the admin api to find all loves
○	`http://localhost:8090/api/ADMIN/loves/search?search[$ne]=1`
20. Give yourself more money!
○	`http://localhost:8000/admin/money`
○	POST to` http://localhost:8000/admin/money?userid=36`

Authentication Vulnerabilities
21. Find the session secret for JWT/sessions.
○	http://localhost:8000/secret.conf
22. Decode a JWT token
○	Jwt.io
23. Brute Force User/Passwords
