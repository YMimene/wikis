# Problem
- we want to use hijri date in the system

# Steps to add hijri date
1. When you create a date field, add another **date field** for the hijri date that should have the same label and ends with "_hijri". <br>
**Example:** if you create a new date field labelled "start_date", then you should add another date field labelled "start_date_hijri"

2. In the doctype js file **on setup**, add the following code:
```
setup: async function(frm) {
	// get the calendar type from the core configuration 
	const calendar_type = await frappe.db.get_single_value("Core Configuration", "calendar_type");
	const is_hijri = calendar_type === "Hijri Om-Qura";

	// get date fields (not hijri)
	const gregorian_date_field_names = []
	Object.entries(frm.fields_dict).forEach(([key, value]) => {
		if (value.df.fieldtype == "Date" && !value.df.fieldname.endsWith("_hijri")) {
			gregorian_date_field_names.push(value.df.fieldname);
		}
		});

	// hide and show depend on the calendar type ans set hijri caledar date
	gregorian_date_field_names.forEach((gregorian_date_field_name) => {
		const hijri_date_field_name = gregorian_date_field_name + "_hijri";

		const gregorian_date_field = frm.fields_dict[gregorian_date_field_name];
		const hijri_date_field = frm.fields_dict[hijri_date_field_name];

		if (is_hijri) {
			gregorian_date_field.df.hidden = 1;
			// gregorian_date_field.df.read_only = 1;
		} else {
			hijri_date_field.df.hidden = 1;
			// hijri_date_field.df.read_only = 1;
		}

		// refresh hijri calendar
		MedadUtils.set_hijri_field_from_gregorian(hijri_date_field, gregorian_date_field);
	});
	frm.refresh_fields();

	// create functions that manipulate dates and assign them to the fields
	gregorian_date_field_names.forEach((gregorian_date_field_name) => {
		const hijri_date_field_name = gregorian_date_field_name + "_hijri";

		// get date and hijri date fields
		const gregorian_date_field = frm.fields_dict[gregorian_date_field_name];
		const hijri_date_field = frm.fields_dict[hijri_date_field_name];

		// create gregorian date function
		const gregorian_date_function = () => {
			if (is_hijri) return;

			const gregorian_date_value = gregorian_date_field.value;
			if (!gregorian_date_value) {
				hijri_date_field.set_value("");
				return;
			}

			// set hijri date field based on gregorian date
			MedadUtils.set_hijri_field_from_gregorian(hijri_date_field, gregorian_date_field);
			frm.refresh_fields();
		}

		// create hijri date function
		const hijri_date_function = () => {
			// to prevent loop
			if (!is_hijri) return;
			
			const hijri_date_value = hijri_date_field.value;
			if (!hijri_date_value) {
				gregorian_date_field.set_value("");
				return;
			}

			// set gregorian date field based on hijri date
			MedadUtils.set_gregorian_field_from_hijri(hijri_date_field, gregorian_date_field);
			frm.refresh_fields();
		}

		// assign function to the fields
		frappe.ui.form.on(frm.doctype, gregorian_date_field_name, gregorian_date_function);
		frappe.ui.form.on(frm.doctype, hijri_date_field_name, hijri_date_function);
	});
}
```

**Example:** if the name of the doctype where you added the date fields called "Student", then you should add the code above like this:
```
frappe.ui.form.on('Student', {
	setup: async function(frm) {
            // add the code above here
	}
});

```
