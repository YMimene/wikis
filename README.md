# Problem
- we want to use hijri date in the system

# Steps to add hijri date
1. When you create a date field, add another **date field** for the hijri date that should have the same label and ends with "_hijri". <br>
**Example:** if you create a new date field labelled "start_date", then you should add another date field labelled "start_date_hijri"

2. In the doctype js file **on setup**, add the following code:
```
setup: function(frm) {
	MedadUtils.enable_hijri_dates(frm);
}
```

**Example:** if the name of the doctype where you added the date fields called "Student", then you should add run the function `MedadUtils.enable_hijri_dates(frm);` like this:
```
frappe.ui.form.on('Student', {
	setup: function(frm) {
            MedadUtils.enable_hijri_dates(frm);
	    // the rest of the code if exists
	}
});

```
