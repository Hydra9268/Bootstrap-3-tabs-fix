# Bootstrap-3-tabs-fix

Placing hyperlinks inside Bootstrap's tabs confuses the code into thinking anything in the href is a tab reference and the error typically appears like:

bootstrap.js:1729 Uncaught Error: Syntax error, unrecognized expression: 

This following fix creates new logic that first checks for the presence of a pound in the href (which we know is native to Bootstrap tabs). If found, the code proceeds as designed. If the pound is not detected, then the code treats this as a regular hyperlink. 

This fix works with jQuery v1.11.3 and Bootstrap v3.1.1.

# Code

Replace "Tab.prototype.show = function() {" in your Bootstrap.js file with the follow function.

    Tab.prototype.show = function() {

        var $this = this.element;
        var $ul = $this.closest('ul:not(.dropdown-menu)');
        var selector = $this.data('target');

        if (!selector) {
            selector = $this.attr('href')
            selector = selector && selector.replace(/.*(?=#[^\s]*$)/, '') //strip for ie7
        }

        var isTabSelector = new RegExp("[\#]").exec(selector);
        isTabSelector = (isTabSelector == null) ? false : true;

        if (isTabSelector) {

            if ($this.parent('li').hasClass('active')) return

            var previous = $ul.find('.active:last a')[0]
            var e = $.Event('show.bs.tab', {
                relatedTarget: previous
            })

            $this.trigger(e)

            if (e.isDefaultPrevented()) return

            var $target = $(selector)

            this.activate($this.parent('li'), $ul)
            this.activate($target, $target.parent(), function() {
                $this.trigger({
                    type: 'shown.bs.tab',
                    relatedTarget: previous
                });
            });
        }
        else
        {
            if (selector.indexOf('http') >= 0)
            {
                window.open(selector);
            }
            else
            {
                window.open(window.location.origin + selector);
            }
        }
    }
