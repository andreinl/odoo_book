Appendix D. Known Inheritances
******************************

::

    class project(osv.osv):
        _name = "project.project"
        _description = "Project"
        _inherits = {'account.analytic.account': "analytic_account_id"}


