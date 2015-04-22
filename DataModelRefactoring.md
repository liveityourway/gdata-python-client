# Introduction #

I recently reworked the data model code to make the process of defining classes easier for developers who are adding new classes. The change has also sped up XML parsing performance and decreased the size of the library.

# Details #

Part of the functionality provided by the library is a conversion mechanism between XML and Python objects. This conversion is carried out in an orderly and consistent way as far as possible.

The way that the conversion to and from XML has been through three major revisons, the most recent on which was released in version 1.3.1. The previous revision, the second, was made available in version 1.0.8. The old model relied on class inheritance with method overloading to ensure that all XML information is converted to and from classes. In the new model, metadata describing the relationship between XML elements and object members is defined using class level members. The conversion methods are generic and rely on this metadata for conversion logic. See an example below.

## Example: gd:when ##

The code for the When object is defined in the `gdata.calendar` module. I chose this class to use an example because the XML for representing When invoves child elements and attributes.

### Third (latest) Version (1.3.1 and later) ###
```
class When(atom.core.XmlElement):
  """The Google Calendar When element"""
  _qname = gdata.data.GDATA_TEMPLATE % 'when'
  reminder = [Reminder]
  start_time = 'startTime'
  end_time = 'endTime'
```

### Second Version (1.0.8 to 1.3.3) ###
```
class When(atom.AtomBase):
  """The Google Calendar When element"""

  _tag = 'when'
  _namespace = gdata.GDATA_NAMESPACE
  _children = atom.AtomBase._children.copy()
  _attributes = atom.AtomBase._attributes.copy()
  _children['{%s}reminder' % gdata.GDATA_NAMESPACE] = ('reminder', [Reminder])
  _attributes['startTime'] = 'start_time'
  _attributes['endTime'] = 'end_time'

  def __init__(self, start_time=None, end_time=None, reminder=None, 
      extension_elements=None, extension_attributes=None, text=None):
    self.start_time = start_time 
    self.end_time = end_time 
    self.reminder = reminder or []
    self.text = text
    self.extension_elements = extension_elements or []
    self.extension_attributes = extension_attributes or {}
```

### First Version (1.0.7 and earlier) ###
```
class When(atom.AtomBase):
  """The Google Calendar When element"""

  def __init__(self, start_time=None, end_time=None, reminder=None,
      extension_elements=None, extension_attributes=None, text=None):
    self.start_time = start_time
    self.end_time = end_time
    self.reminder = reminder or []
    self.text = text
    self.extension_elements = extension_elements or []
    self.extension_attributes = extension_attributes or {}

  def _TransferToElementTree(self, element_tree):
    if self.start_time:
      element_tree.attrib['startTime'] = self.start_time
    if self.end_time:
      element_tree.attrib['endTime'] = self.end_time
    for a_reminder in self.reminder:
      a_reminder._BecomeChildElement(element_tree)
    atom.AtomBase._TransferToElementTree(self, element_tree)
    element_tree.tag = gdata.GDATA_TEMPLATE % 'when'
    return element_tree

  def _TakeChildFromElementTree(self, child, element_tree):
    if child.tag == '{%s}%s' % (gdata.GDATA_NAMESPACE, 'reminder'):
      self.reminder.append(_ReminderFromElementTree(child))
      element_tree.remove(child)
    else:
      atom.AtomBase._TakeChildFromElementTree(self, child, element_tree)

  def _TakeAttributeFromElementTree(self, attribute, element_tree):
    if attribute == 'startTime':
      self.start_time = element_tree.attrib[attribute]
      del element_tree.attrib[attribute]
    elif attribute == 'endTime':
      self.end_time = element_tree.attrib[attribute]
      del element_tree.attrib[attribute]
    else:
      atom.AtomBase._TakeAttributeFromElementTree(self, attribute,
          element_tree)
```