Problem:
- we want to use hijri date in the system

Steps to add hijri date:
1. When you create a date field, add another field for the hijri date that should have the same label and ends with "_hijri".
Example: if you create a new date field labelled "start_date", then you should add another date field labelled "start_date_hijri"

2. In the doctype js file on setup, add the following code:
```
// get the calendar type from the core configuration 
const calendar_type = await frappe.db.get_single_value("Core Configuration", "calendar_type");
is_hijri = calendar_type === "Hijri Om-Qura";

// get date fields (not hijri)
date_field_names = []
Object.entries(frm.fields_dict).forEach(([key, value]) => {
	if (value.df.fieldtype == "Date" && !value.df.fieldname.endsWith("_hijri")) {
		date_field_names.push(value.df.fieldname);
	}
	});

// hide and show depend on the calendar type ans set hijri caledar date
date_field_names.forEach((date_field_name) => {
	const hijri_date_field_name = date_field_name + "_hijri";

	const date_field = frm.fields_dict[date_field_name];
	const hijri_date_field = frm.fields_dict[hijri_date_field_name];

	if (is_hijri) {
		date_field.df.hidden = 1;
		set_hijri_calendar_date(hijri_date_field.value, hijri_date_field.cal);
	} else {
		hijri_date_field.df.hidden = 1;
	}
});
frm.refresh_fields();

// create functions that manipulate dates and assign them to the fields
date_field_names.forEach((date_field_name) => {
	const hijri_date_field_name = date_field_name + "_hijri";

	// get date and hijri date fields
	const date_field = frm.fields_dict[date_field_name];
	const hijri_date_field = frm.fields_dict[hijri_date_field_name];

	// create date function
	const date_function = () => {
		if (is_hijri) return;

		const date_value = date_field.value;
		if (!date_value) {
			hijri_date_field.set_value("");
			return;
		}

		// get gregorian date
		const date_splitted = date_value.split("-");
		const day = parseInt(date_splitted[2]);
		const month = parseInt(date_splitted[1]);
		const year = parseInt(date_splitted[0]);

		// convert from gregorian to hijri
		const date_value_hijri  = from_gregorian_to_hijri(day, month, year, true);
		
		// set hijri date in the field and calendar
		hijri_date_field.set_value(date_value_hijri);
		set_hijri_calendar_date(date_value_hijri, hijri_date_field.cal);

		frm.refresh_fields();
	}

	// create hijri date function
	const hijri_date_function = () => {
		// to prevent loop
		if (!is_hijri) return;

		// get hijri date and convert it to show it
		const date_value = from_hijri_to_gregorian(hijri_date_field.cal.getDate(), true);
		date_field.set_value(date_value);
	}

	// assign function to the fields
	frappe.ui.form.on('Academic Service', date_field_name, date_function);
	frappe.ui.form.on('Academic Service', hijri_date_field_name, hijri_date_function);
});
```

Example: if the name of the doctype where you added the date fields called "Student", then you should add the code above like this:
```
frappe.ui.form.on('Student', {
	setup: async function(frm) {
            // add the code above here
	}
});

```
3. Add the helper functions in each doctype js file or add them to a separate file and call them from the current file
```
function from_hijri_to_gregorian(date, is_reversed = false) {
	// get date fields
	date = date.getGregorianDate();
	const day = date.getDate();
	const month = date.getMonth() + 1;
	const year = date.getFullYear();

	return get_formatted_date(day, month, year, is_reversed);
}


function from_gregorian_to_hijri(day, month, year, is_reversed = false) {
	let hijri_date = new Intl.DateTimeFormat('en-US', {
		calendar: 'islamic-umalqura',
	}).format(Date.UTC(year, month - 1, day));

	hijri_date = hijri_date.split(" ")[0].split("/");

	day = hijri_date[1];
	month = hijri_date[0];
	year = hijri_date[2];

	return get_formatted_date(day, month, year, is_reversed);
}


function get_formatted_date(day, month, year, is_reversed = false) {
	// convert to string
	day = day.toString();
	month = month.toString();
	year = year.toString();

	// add leading 0
    if (month.length < 2) month = "0" + month;
    if (day.length < 2) day = "0" + day;
    
	if (is_reversed) 
        return `${year}-${month}-${day}`;
    return `${day}-${month}-${year}`;
}


function set_hijri_calendar_date(hijri_date, calendar) {
	if (!hijri_date) return;
	const hijri_date_splitted = hijri_date.split("-");

	const day = hijri_date_splitted[2];
	const month = parseInt(hijri_date_splitted[1]) - 1;
	const year = hijri_date_splitted[0];

	calendar.setDate(year, month, day);
}
```
