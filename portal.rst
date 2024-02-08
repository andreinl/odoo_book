Portal related Information
***************************

Templates
=========

Input Date::

    <input id="tsheet_date" name="tsheet_date" class="input-group date"
                           type="date" data-date-format="YYYY-MM-DD" t-attf-class="form-control #{'form-control-sm' if form_small else ''}"
                           required="required" />

Input DateTime::

    <input id="tsheet_date_start" name="tsheet_date_start" class="input-group datetime"
                           type="datetime-local" data-date-format="YYYY-MM-DD HH:mm:ss" t-attf-class="form-control #{'form-control-sm' if form_small else ''}"
                           required="required" />


Controllers
===========

Request Methods::

    @http.route(['/my/timesheet',], type='http', auth="user", website=True, methods=['GET', 'POST'])
    def portal_new_tsheet(self, **kwargs):
        if request.httprequest.environ['REQUEST_METHOD'] == 'POST':
            pass
        else:
            pass


