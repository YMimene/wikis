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

# How to enable hijri dates in child tables
1. for each date field, add a hijri date field in the child table doctype, with a name that ends with "_hijri"
2. for each date field, add the following code to the js file:
```
const cahed_child_hijri_dates = {}

frappe.ui.form.on(child_table_doctype, hijri_date_field_name, function(frm, cdt, cdn){
	MedadUtils.child_on_date_hijri(frm, cdt, cdn, gregorian_date_field_name, cahed_child_hijri_dates)
});

frappe.ui.form.on(child_table_doctype, gregorian_date_field_name, function(frm, cdt, cdn){
	MedadUtils.child_on_date_gregorian(frm, cdt, cdn, gregorian_date_field_name, cahed_child_hijri_dates)
});


frappe.ui.form.on(Doctype, {
	onload_post_render: function (frm) {
		MedadUtils.doctype_onload_post_render(frm, child_table_doctype, child_table_field_name, gregorian_date_field_name, cahed_child_hijri_dates);
	},
	before_load: function (frm) {
		MedadUtils.doctype_before_load(frm, child_table_doctype, gregorian_date_field_name, child_table_field_name);
	},
});

```
where:
- gregorian_date_field_name: the date field name in the child table
- hijri_date_field_name: the hijri date name in the child table
- child_table_doctype: the name of the child doctype
- child_table_field_name: the name of the child table field in the parent doctype

Here is an example, of using the above code where there are multiple date fields in the child table from the **Time Ticket Group** doctype js file:
```
const cahed_child_hijri_dates = {}

// start date functions
frappe.ui.form.on('Time Ticket Group Period Item', "period_start_date", function(frm, cdt, cdn){
	MedadUtils.child_on_date_gregorian(frm, cdt, cdn, "period_start_date", cahed_child_hijri_dates)
});

frappe.ui.form.on('Time Ticket Group Period Item', "period_start_date_hijri", function(frm, cdt, cdn){
	MedadUtils.child_on_date_hijri(frm, cdt, cdn, "period_start_date", cahed_child_hijri_dates)
});

// end date functions
frappe.ui.form.on('Time Ticket Group Period Item', "period_end_date", function(frm, cdt, cdn){
	MedadUtils.child_on_date_gregorian(frm, cdt, cdn, "period_end_date", cahed_child_hijri_dates)
});

frappe.ui.form.on('Time Ticket Group Period Item', "period_end_date_hijri", function(frm, cdt, cdn){
	MedadUtils.child_on_date_hijri(frm, cdt, cdn, "period_end_date", cahed_child_hijri_dates)
});


frappe.ui.form.on('Time Ticket Group', {
    onload_post_render: function (frm) {
        MedadUtils.doctype_onload_post_render(frm, "Time Ticket Group Period Item", "periods", "period_start_date", cahed_child_hijri_dates);
        MedadUtils.doctype_onload_post_render(frm, "Time Ticket Group Period Item", "periods", "period_end_date", cahed_child_hijri_dates);
    },
    before_load: async function (frm) {
        MedadUtils.doctype_before_load(frm, "Time Ticket Group Period Item", "period_start_date", "periods");
        MedadUtils.doctype_before_load(frm, "Time Ticket Group Period Item", "period_end_date", "periods");
    },

});

```

# How to handle hijri dates in reports
for the reports there are two cases:

### 1. when the date field is in the filters
in this case, we will define our filters after identifying the type of the calendar. So we will make a function that get the type of the calendar and based on that, we will show gregorian date filters or hijri date filters, here is a commented example on how the **.js** file will look like:
```
// start with no filters
frappe.query_reports["Message Log"] = {
	"filters": []
};

// define the function that will update the filters based on the calendar type
async function show_filters_based_on_calendar_type() {
	// get the type of the calendar
	const calendar_type = (await frappe.db.get_doc("Core Configuration")).calendar_type;
	const is_hijri = calendar_type !== "Gregorian";
	
	// update the filters based on the type of the calendar
	frappe.query_reports["Message Log"] = {
		"filters": [
			{
				"fieldname": "from_date",
				"label": __("From"),
				"fieldtype": "Date",
				// hide or unhide based on the calendar type
				"hidden": is_hijri ? 1 : 0,
				"reqd": is_hijri ? 0 : 1,
			},
			{
				"fieldname": "from_date_hijri",
				"label": __("From"),
				"fieldtype": "Date",
				"hidden": is_hijri ? 0 : 1,
				"reqd": is_hijri ? 1 : 0,
			},
			
		]
	};
	// re-load the report to show the filters based on the calendar type
	frappe.query_report.load_report()
}

// don't forget to call the function 
show_filters_based_on_calendar_type();
```

### 2. when the date is in the report results
in this case, we will convert the date to hijri if the calendar is hijri, else we show the gregorian date, to do that:
- we use the is_hijri() function from helpers module in the medad_sis_core to get if the hijri calendar is enabled
- we use the functions: **to_gregorian()** and **to_hijri()** to convert between gregorian and hijri

You find a full example with both cases is the **Trainer Short Courses** report.

# How to handle datetime fields
Note that we decided that we do not use datetime fields anymore, and if we need date and time we use two seprated fields: date field and time field.
and the following explanation is just for the existing datetime fields (you will not use it mostly)

For the existing datetime fields, we used the following approch to solve the problem:
- for each datetime field, we added three other fields: date field, hijri date field and time field
- we kept the datetime field hidden and we change it whenever one of the other three fields got changed and also we check the values of 3 fields in the before_save in the python file. You can find this case in the **Financial Transaction** doctype

- same when the datetime time appears in the child table, we create three other fields (date, hijri date, time) for each datetime field then, in the before_save (in the python file) we check the values: change the value of the datetime field based on the date, hijri date and time fields and the inverse. You can find a detailed example in the **Time Ticket Group** doctype.

