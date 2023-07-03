Appendix E. Patching
********************


Create a patch to submit a solution to Launchpad
================================================

Create file diff like this (patch_webdav.diff)::

    Index: /dati/lp/Virtual_Openerp_61_ocb/addons-ocb/document_webdav/dav_fs.py
    --- document_webdav/dav_fs.py	2011-12-19 16:54:40 +0000
    +++ document_webdav/dav_fs.py	2013-12-26 09:35:45 +0000
    @@ -172,6 +172,7 @@
             self.parent = parent
             self.baseuri = parent.baseuri
             self.verbose = verbose
    +        self.nods = {}
     
         def get_propnames(self, uri):
             props = self.PROPS
    @@ -475,8 +476,11 @@
         def uri2object(self, cr, uid, pool, uri):
             if not uid:
                 return None
    -        context = self.reduce_useragent()
    -        return pool.get('document.directory').get_object(cr, uid, uri, context=context)
    +        path = '/'.join(uri)
    +        if not uri or not path in self.nods:
    +            context = self.reduce_useragent()
    +            self.nods[path] = pool.get('document.directory').get_object(cr, uid, uri, context=context)
    +        return self.nods[path]
     
         def get_data(self,uri, rrange=None):
             self.parent.log_message('GET: %s' % uri)
             
Change the first line::

    Index: document_webdav/dav_fs.py
    --- document_webdav/dav_fs.py	2011-12-19 16:54:40 +0000
    +++ document_webdav/dav_fs.py	2013-12-26 09:35:45 +0000
    
    
Apply a patch
=============

Place a patch into directory that contains a document_webdav folder and type::

    patch -p0 < patch_webdav.diff
    

