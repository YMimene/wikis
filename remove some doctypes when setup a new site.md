# Remove some doctypes when setup a new site

### using hooks (automatic)
- add a hook in the core app that check if we deleted the unecessary doctypes
  - we can add a hidden check in the core configuration to indicate if we removed the changes or not (to avoid removing doctypes each time).
- we can use the after_installation hook that do the requirement after installing the core app in the site
  - [resource](https://frappeframework.com/docs/v13/user/en/python-api/hooks#install-hooks)

### using a button (manual)
- we can add a button in the core configuration for example and when the admin click the button, we delete the unecessary doctypes


