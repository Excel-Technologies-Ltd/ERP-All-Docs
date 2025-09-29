```javascript
frappe.ui.form.on('Meeting Room Booking', {
    onload:function(frm){
//      var credit_row = frm.fields_dict.existing_meetings.grid.add_new_row();
// frappe.model.set_value(credit_row.doctype, credit_row.name, "title", "test");
frm.get_field("existing_meetings").grid.cannot_add_rows = true;
frm.fields_dict['existing_meetings'].grid.wrapper.find('.btn-open-row').hide()



    },
  excel_branch:function(frm){
      existing_meeting(frm)
  },
  meeting_room:function(frm){
      existing_meeting(frm)
  },
  start_date:function(frm){
      existing_meeting(frm)
  },
  end_date:function(frm){
      existing_meeting(frm)
  },



    start_time:function(frm){
        if(frm.doc.start_time && frm.doc.duration){
            console.log("true")
            const end_time=calculateEndTime(frm.doc.start_time,frm.doc.duration)
             frm.set_value("end_time", end_time);
              frm.reload_doc("end_time")
        }
    },

	duration:function(frm){
	    const start_time=frm.doc.start_time
	    console.log("dkcedg")
	    if(!start_time){
	        frappe.msgprint("First Enter Start Time")
	        frm.set_value("duration","")
	        frm.reload_doc("duration")
	    }else{
	        const duration=frm.doc.duration
	        const end_time=calculateEndTime(start_time,duration)
	                    if (duration > 10 * 3600) {
                frappe.msgprint("Meeting duration should not exceed 4 hours");
                frm.set_value("duration", ""); // Set duration to 4 hours
                frm.reload_doc("duration");
            } else {
                frm.set_value("end_time", end_time);
            }
	       
	    }
	},
	recursion:function(frm){
	    if(frm.doc.recursion==1){
	        frm.set_df_property("end_date","read_only" ,0)
	    }else{
	        frm.set_df_property("end_date","read_only" ,1) 
	    }
	},
    excel_branch: function(frm) {
        const branch = frm.doc.excel_branch;
     
            const floor = frm.doc.excel_floor;
            frm.set_query("meeting_room", function() {
                return {
                    filters: {
                        excel_branch: branch
                    }
                };
            });
        }

 




})

// frappe.ui.form.on('Meeting Room Booking', 'refresh', function(frm) {
//     frappe.call({
//         'method': 'frappe.client.get_list',
//         'args': {
//             'doctype': 'Meeting',
//             'columns': ['meeting_date', "excel_start_time", "excel_end_time"]
//         },
//         'callback': function(res) {
//             console.log(res)
//             let template = "<h1>table</h1>";
//             frm.set_df_property('html_fieldname', 'options', frappe.render(template, ));
//             frm.refresh_field('html_fieldname');
//         }
//     })
// });

function calculateEndTime(startTimeStr, durationSeconds) {
  // Parse the start time string
  const startTimeComponents = startTimeStr.split(":");
  const startTime = new Date(0, 0, 0, startTimeComponents[0], startTimeComponents[1], startTimeComponents[2]);

  // Convert start time components to seconds
  const startTimeSeconds = startTime.getHours() * 3600 + startTime.getMinutes() * 60 + startTime.getSeconds();

  // Calculate end time in seconds
  const endTimeSeconds = startTimeSeconds + durationSeconds;

  // Convert end time seconds back to hours, minutes, and seconds
  const endTimeHours = Math.floor(endTimeSeconds / 3600);
  const endTimeMinutes = Math.floor((endTimeSeconds % 3600) / 60);
  const endTimeSecondsRemainder = endTimeSeconds % 60;

  // Format the end time
  const endTimeStr = `${String(endTimeHours).padStart(2, '0')}:${String(endTimeMinutes).padStart(2, '0')}:${String(endTimeSecondsRemainder).padStart(2, '0')}`;

  return endTimeStr;
}



const getSingleDoc=(Doctype,name,field)=>{
    frappe.call({
        method: 'frappe.client.get_value',
        args: {
            'doctype': Doctype,
            'filters': {'name': name},
            'fieldname': field
        },
        callback: function(r) {
            if (r.message) {
                console.log(r)
               return r
            }
        }
    });
}
function existing_meeting(frm){
    if(frm.doc.excel_branch && frm.doc.meeting_room && frm.doc.start_date &&  frm.doc.end_date){
        const branch=frm.doc.excel_branch
         const meeting_room=frm.doc.meeting_room
         const start_date=frm.doc.start_date
         const end_date=frm.doc.end_date
       frm.fields_dict.existing_meetings.grid.remove_all();
        console.log("workimh")
        frappe.call({
    method: 'frappe.client.get_list',
    args: {
        doctype: 'Meeting',
        filters: [
            ['excel_branch', '=', branch],
            ['meeting_room', '=', meeting_room],
            ['meeting_date', '>=', start_date],
             ['meeting_date', '<=', end_date],
            
           
        ],
        fields: [ 'title', "meeting_date","excel_start_time","excel_end_time"],
    },
    callback: e => {
        const datas=e.message
        console.log(datas)
        datas.map(data => {
             var credit_row = frm.fields_dict.existing_meetings.grid.add_new_row();
frappe.model.set_value(credit_row.doctype, credit_row.name, "title", data.title);
frappe.model.set_value(credit_row.doctype, credit_row.name, "date", data.meeting_date);
frappe.model.set_value(credit_row.doctype, credit_row.name, "start_time", data.excel_start_time);
frappe.model.set_value(credit_row.doctype, credit_row.name, "end_time", data.excel_end_time);
        })
    }
})
    }
}
