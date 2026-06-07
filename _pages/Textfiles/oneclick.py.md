# find all rss feeds within one click of a URL

__VERSION__ = "0.01"
__DATE__ = "2003-12-22"
__AUTHOR__ = "Phillip Pearson"
__COPYRIGHT__ = "Copyright (C) 2003 Phillip Pearson"
__LICENSE__ = "Python"
__HISTORY__ = """

0.01 - PP - initial release

"""

import sgmllib, urllib, rssfinder, sys

class parser(sgmllib.SGMLParser):
    def __init__(self, root_url):
        sgmllib.SGMLParser.__init__(self)
        self.root_url = root_url
        self.found_urls = {}
        
    def do_a(self, attrs):
        for k,v in attrs:
            if k.lower() == 'href':
                url = urllib.basejoin(self.root_url, v)
                if not url.startswith(self.root_url):
                    self.found_urls[url] = 1
                return

def findall(url, excl):
    txt = urllib.urlopen(url).read()
    p = parser(url)
    try:
        p.feed(txt)
        p.close()
    except:
        htmlfn = 'bad.html'
        open(htmlfn, 'wt').write(txt)
        print "exception parsing html; saved as %s" % htmlfn
        raise

    def link_ok(link):
        if excl and link.find(excl) != -1:
            return 0
        if not link.startswith('http://'):
            return 0
        for bad_start in ('http://locahost', 'http://127.0.0.1'):
            if link.startswith(bad_start):
                return 0
        return 1
    
    links = [x for x in p.found_urls.keys() if link_ok(x)]
    links.sort()
    print "Found the following links:"
    for link in links:
        print link
    print
    print "Finding RSS feeds ..."
    for k in links:
        print "URL %s:" % k
        try:
            for f in rssfinder.getFeeds(k):
                print "\t%s" % f
        except IOError:
            print "\t(failed to fetch)"

if __name__=='__main__':
    # Syntax:   python oneclick.py [url] [exclusion-pattern]
    #
    # e.g. python oneclick.py http://scripting.com/ archive.scripting.com
    #           analyses scripting.com, but avoids scripting.com archive pages
    #
    # e.g. python oneclick.py http://pyds.muensterland.org/
    #           analyses georg's blog
    #
    # e.g. python oneclick.py
    #           analyses my blog
    excl = None
    if len(sys.argv) > 1:
        url = sys.argv[1] # url to check
        if len(sys.argv) > 2:
            excl = sys.argv[2] # exclusion pattern
    else:
        url = "http://www.myelin.co.nz/post/"
    findall(url, excl)
