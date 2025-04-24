import { useState } from 'react'
import { Button } from "/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "/components/ui/card"
import { Input } from "/components/ui/input"
import { Label } from "/components/ui/label"
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "/components/ui/select"
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "/components/ui/table"
import { Calendar as CalendarIcon, Bell, Users, Plus, X } from "lucide-react"
import { format, addMonths, subMonths, startOfMonth, endOfMonth, eachDayOfInterval, isSameMonth, isSameDay } from 'date-fns'

type Document = {
  id: string
  title: string
  category: string
  description: string
  date: string
}

type User = {
  email: string
  password: string
}

type CalendarEvent = {
  id: string
  title: string
  date: string
  description: string
}

type Notice = {
  id: string
  title: string
  date: string
}

export default function DocumentManagementApp() {
  const [isLoggedIn, setIsLoggedIn] = useState(false)
  const [activeTab, setActiveTab] = useState('dashboard')
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [newUserEmail, setNewUserEmail] = useState('')
  const [newUserPassword, setNewUserPassword] = useState('')
  const [documentTitle, setDocumentTitle] = useState('')
  const [documentCategory, setDocumentCategory] = useState('')
  const [documentDescription, setDocumentDescription] = useState('')
  const [searchTerm, setSearchTerm] = useState('')
  const [users, setUsers] = useState<User[]>([
    { email: 'wincat@gmail.com', password: '1234567' }
  ])
  const [documents, setDocuments] = useState<Document[]>([
    {
      id: '1',
      title: 'Sample Judgment',
      category: 'Judgment',
      description: 'Final ruling on case #12345',
      date: '2023-05-15'
    }
  ])
  const [calendarEvents, setCalendarEvents] = useState<CalendarEvent[]>([
    { id: '1', title: 'Hearing for Case #123', date: '2023-06-20', description: 'Courtroom 5' },
    { id: '2', title: 'Meeting with Client', date: '2023-06-22', description: 'Office' }
  ])
  const [notices] = useState<Notice[]>([
    { id: '1', title: 'Court Holiday Notice', date: '2023-06-15' },
    { id: '2', title: 'New Filing Requirements', date: '2023-06-10' }
  ])
  const [currentMonth, setCurrentMonth] = useState(new Date())
  const [selectedDate, setSelectedDate] = useState<Date | null>(null)
  const [newEventTitle, setNewEventTitle] = useState('')
  const [newEventDescription, setNewEventDescription] = useState('')
  const [showEventForm, setShowEventForm] = useState(false)

  // Calendar navigation functions
  const nextMonth = () => setCurrentMonth(addMonths(currentMonth, 1))
  const prevMonth = () => setCurrentMonth(subMonths(currentMonth, 1))

  // Get days for current month view
  const monthStart = startOfMonth(currentMonth)
  const monthEnd = endOfMonth(currentMonth)
  const monthDays = eachDayOfInterval({ start: monthStart, end: monthEnd })

  const handleLogin = (e: React.FormEvent) => {
    e.preventDefault()
    const user = users.find(u => u.email === email && u.password === password)
    if (user) {
      setIsLoggedIn(true)
    } else {
      alert('Invalid credentials')
    }
  }

  const handleLogout = () => {
    setIsLoggedIn(false)
    setEmail('')
    setPassword('')
  }

  const handleAddUser = (e: React.FormEvent) => {
    e.preventDefault()
    if (newUserEmail && newUserPassword) {
      setUsers([...users, { email: newUserEmail, password: newUserPassword }])
      setNewUserEmail('')
      setNewUserPassword('')
      alert('User created successfully')
    }
  }

  const handleUploadDocument = (e: React.FormEvent) => {
    e.preventDefault()
    if (documentTitle && documentCategory) {
      const newDocument: Document = {
        id: Date.now().toString(),
        title: documentTitle,
        category: documentCategory,
        description: documentDescription,
        date: new Date().toISOString().split('T')[0]
      }
      setDocuments([newDocument, ...documents])
      setDocumentTitle('')
      setDocumentCategory('')
      setDocumentDescription('')
      alert('Document uploaded successfully')
    }
  }

  const handleDeleteDocument = (id: string) => {
    setDocuments(documents.filter(doc => doc.id !== id))
  }

  const handleDateClick = (day: Date) => {
    setSelectedDate(day)
    setShowEventForm(true)
    setNewEventTitle('')
    setNewEventDescription('')
  }

  const handleAddEvent = (e: React.FormEvent) => {
    e.preventDefault()
    if (selectedDate && newEventTitle) {
      const newEvent: CalendarEvent = {
        id: Date.now().toString(),
        title: newEventTitle,
        description: newEventDescription,
        date: format(selectedDate, 'yyyy-MM-dd')
      }
      setCalendarEvents([...calendarEvents, newEvent])
      setNewEventTitle('')
      setNewEventDescription('')
      setShowEventForm(false)
    }
  }

  const handleDeleteEvent = (id: string) => {
    setCalendarEvents(calendarEvents.filter(event => event.id !== id))
  }

  const filteredDocuments = documents.filter(doc =>
    doc.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
    doc.category.toLowerCase().includes(searchTerm.toLowerCase())
  )

  const eventsForSelectedDate = selectedDate 
    ? calendarEvents.filter(event => 
        isSameDay(new Date(event.date), selectedDate)
      )
    : []

  if (!isLoggedIn) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-100">
        <Card className="w-full max-w-md">
          <CardHeader>
            <CardTitle className="text-2xl font-bold text-center">Login</CardTitle>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleLogin} className="space-y-4">
              <div className="space-y-2">
                <Label htmlFor="email">Email</Label>
                <Input
                  id="email"
                  type="email"
                  value={email}
                  onChange={(e) => setEmail(e.target.value)}
                  placeholder="wincat@gmail.com"
                  required
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="password">Password</Label>
                <Input
                  id="password"
                  type="password"
                  value={password}
                  onChange={(e) => setPassword(e.target.value)}
                  placeholder="1234567"
                  required
                />
              </div>
              <Button type="submit" className="w-full">
                Login
              </Button>
            </form>
          </CardContent>
        </Card>
      </div>
    )
  }

  return (
    <div className="flex min-h-screen bg-white">
      {/* Sidebar */}
      <div className="w-64 bg-blue-900 text-white p-4">
        <h1 className="text-xl font-bold mb-8">Admin Panel</h1>
        <nav className="space-y-2">
          <button
            onClick={() => setActiveTab('dashboard')}
            className={`block w-full text-left p-2 rounded ${activeTab === 'dashboard' ? 'bg-orange-500' : 'hover:bg-blue-800'}`}
          >
            Dashboard
          </button>
          <button
            onClick={() => setActiveTab('calendar')}
            className={`block w-full text-left p-2 rounded ${activeTab === 'calendar' ? 'bg-orange-500' : 'hover:bg-blue-800'}`}
          >
            Calendar Events
          </button>
          <button
            onClick={() => setActiveTab('notices')}
            className={`block w-full text-left p-2 rounded ${activeTab === 'notices' ? 'bg-orange-500' : 'hover:bg-blue-800'}`}
          >
            Notices
          </button>
          <button
            onClick={() => setActiveTab('documents')}
            className={`block w-full text-left p-2 rounded ${activeTab === 'documents' ? 'bg-orange-500' : 'hover:bg-blue-800'}`}
          >
            Court Documents
          </button>
          <button
            onClick={() => setActiveTab('users')}
            className={`block w-full text-left p-2 rounded ${activeTab === 'users' ? 'bg-orange-500' : 'hover:bg-blue-800'}`}
          >
            Users
          </button>
          <button
            onClick={() => setActiveTab('settings')}
            className={`block w-full text-left p-2 rounded ${activeTab === 'settings' ? 'bg-orange-500' : 'hover:bg-blue-800'}`}
          >
            Settings
          </button>
          <button
            onClick={handleLogout}
            className="block w-full text-left p-2 rounded hover:bg-blue-800"
          >
            Logout
          </button>
        </nav>
      </div>

      {/* Main Content */}
      <div className="flex-1 p-8">
        {activeTab === 'dashboard' && (
          <>
            <div className="flex justify-between items-center mb-8">
              <h2 className="text-2xl font-bold">Admin Dashboard</h2>
              <div className="flex space-x-4">
                <Input
                  type="text"
                  placeholder="Search..."
                  className="w-64"
                  value={searchTerm}
                  onChange={(e) => setSearchTerm(e.target.value)}
                />
                <Button variant="outline">Super Admin</Button>
              </div>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
              <Card>
                <CardHeader className="flex flex-row items-center justify-between pb-2">
                  <CardTitle className="text-sm font-medium">Calendar Events</CardTitle>
                  <CalendarIcon className="h-4 w-4 text-muted-foreground" />
                </CardHeader>
                <CardContent>
                  <div className="text-2xl font-bold">{calendarEvents.length}</div>
                  <p className="text-xs text-muted-foreground">Upcoming events</p>
                </CardContent>
              </Card>

              <Card>
                <CardHeader className="flex flex-row items-center justify-between pb-2">
                  <CardTitle className="text-sm font-medium">Notices</CardTitle>
                  <Bell className="h-4 w-4 text-muted-foreground" />
                </CardHeader>
                <CardContent>
                  <div className="text-2xl font-bold">{notices.length}</div>
                  <p className="text-xs text-muted-foreground">Active notices</p>
                </CardContent>
              </Card>

              <Card>
                <CardHeader className="flex flex-row items-center justify-between pb-2">
                  <CardTitle className="text-sm font-medium">Users</CardTitle>
                  <Users className="h-4 w-4 text-muted-foreground" />
                </CardHeader>
                <CardContent>
                  <div className="text-2xl font-bold">{users.length}</div>
                  <p className="text-xs text-muted-foreground">Registered users</p>
                </CardContent>
              </Card>
            </div>
          </>
        )}

        {activeTab === 'calendar' && (
          <>
            <div className="flex justify-between items-center mb-8">
              <h2 className="text-2xl font-bold">Calendar Events</h2>
            </div>

            <Card className="mb-8">
              <CardHeader className="flex flex-row items-center justify-between">
                <CardTitle>
                  {format(currentMonth, 'MMMM yyyy')}
                </CardTitle>
                <div className="flex space-x-2">
                  <Button variant="outline" size="sm" onClick={prevMonth}>
                    Previous
                  </Button>
                  <Button variant="outline" size="sm" onClick={nextMonth}>
                    Next
                  </Button>
                </div>
              </CardHeader>
              <CardContent>
                <div className="grid grid-cols-7 gap-1">
                  {['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'].map(day => (
                    <div key={day} className="text-center font-medium text-sm py-2">
                      {day}
                    </div>
                  ))}
                  {monthDays.map(day => {
                    const dayEvents = calendarEvents.filter(event => 
                      isSameDay(new Date(event.date), day)
                    )
                    return (
                      <div
                        key={day.toString()}
                        onClick={() => handleDateClick(day)}
                        className={`p-2 border rounded min-h-16 cursor-pointer hover:bg-gray-50 ${
                          !isSameMonth(day, currentMonth) ? 'text-gray-400' : ''
                        } ${
                          selectedDate && isSameDay(day, selectedDate) ? 'bg-blue-50 border-blue-200' : ''
                        }`}
                      >
                        <div className="text-right">{format(day, 'd')}</div>
                        {dayEvents.length > 0 && (
                          <div className="mt-1 space-y-1">
                            {dayEvents.slice(0, 2).map(event => (
                              <div 
                                key={event.id} 
                                className="text-xs p-1 bg-blue-100 text-blue-800 rounded truncate"
                              >
                                {event.title}
                              </div>
                            ))}
                            {dayEvents.length > 2 && (
                              <div className="text-xs text-gray-500">+{dayEvents.length - 2} more</div>
                            )}
                          </div>
                        )}
                      </div>
                    )
                  })}
                </div>
              </CardContent>
            </Card>

            {showEventForm && selectedDate && (
              <Card className="mb-8">
                <CardHeader className="flex flex-row items-center justify-between">
                  <CardTitle>
                    Add Event for {format(selectedDate, 'MMMM d, yyyy')}
                  </CardTitle>
                  <Button 
                    variant="ghost" 
                    size="sm" 
                    onClick={() => setShowEventForm(false)}
                  >
                    <X className="h-4 w-4" />
                  </Button>
                </CardHeader>
                <CardContent>
                  <form onSubmit={handleAddEvent} className="space-y-4">
                    <div className="space-y-2">
                      <Label htmlFor="event-title">Event Title</Label>
                      <Input
                        id="event-title"
                        value={newEventTitle}
                        onChange={(e) => setNewEventTitle(e.target.value)}
                        required
                      />
                    </div>
                    <div className="space-y-2">
                      <Label htmlFor="event-description">Description</Label>
                      <Input
                        id="event-description"
                        value={newEventDescription}
                        onChange={(e) => setNewEventDescription(e.target.value)}
                      />
                    </div>
                    <Button type="submit" className="flex items-center gap-1">
                      <Plus className="h-4 w-4" />
                      Add Event
                    </Button>
                  </form>
                </CardContent>
              </Card>
            )}

            {selectedDate && (
              <Card>
                <CardHeader>
                  <CardTitle>
                    Events for {format(selectedDate, 'MMMM d, yyyy')}
                  </CardTitle>
                </CardHeader>
                <CardContent>
                  {eventsForSelectedDate.length > 0 ? (
                    <Table>
                      <TableHeader>
                        <TableRow>
                          <TableHead>Title</TableHead>
                          <TableHead>Description</TableHead>
                          <TableHead>Actions</TableHead>
                        </TableRow>
                      </TableHeader>
                      <TableBody>
                        {eventsForSelectedDate.map(event => (
                          <TableRow key={event.id}>
                            <TableCell>{event.title}</TableCell>
                            <TableCell>{event.description}</TableCell>
                            <TableCell>
                              <Button variant="ghost" size="sm">Edit</Button>
                              <Button 
                                variant="ghost" 
                                size="sm"
                                onClick={() => handleDeleteEvent(event.id)}
                              >
                                Delete
                              </Button>
                            </TableCell>
                          </TableRow>
                        ))}
                      </TableBody>
                    </Table>
                  ) : (
                    <p>No events scheduled for this day</p>
                  )}
                </CardContent>
              </Card>
            )}
          </>
        )}

        {activeTab === 'documents' && (
          <>
            <div className="flex justify-between items-center mb-8">
              <h2 className="text-2xl font-bold">Court Documents</h2>
              <div className="flex space-x-4">
                <Input
                  type="text"
                  placeholder="Search documents..."
                  className="w-64"
                  value={searchTerm}
                  onChange={(e) => setSearchTerm(e.target.value)}
                />
              </div>
            </div>

            <Card className="mb-8">
              <CardHeader>
                <CardTitle>Upload New Document</CardTitle>
              </CardHeader>
              <CardContent>
                <p className="mb-4">Upload court documents such as judgments, orders, and notifications.</p>
                <form onSubmit={handleUploadDocument} className="space-y-4">
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div className="space-y-2">
                      <Label htmlFor="title">Document Title</Label>
                      <Input
                        id="title"
                        value={documentTitle}
                        onChange={(e) => setDocumentTitle(e.target.value)}
                        required
                      />
                    </div>
                    <div className="space-y-2">
                      <Label htmlFor="category">Category</Label>
                      <Select
                        value={documentCategory}
                        onValueChange={setDocumentCategory}
                        required
                      >
                        <SelectTrigger>
                          <SelectValue placeholder="Select category" />
                        </SelectTrigger>
                        <SelectContent>
                          <SelectItem value="Judgment">Judgment</SelectItem>
                          <SelectItem value="Order">Order</SelectItem>
                          <SelectItem value="Notification">Notification</SelectItem>
                          <SelectItem value="Other">Other</SelectItem>
                        </SelectContent>
                      </Select>
                    </div>
                  </div>
                  <div className="space-y-2">
                    <Label htmlFor="description">Description</Label>
                    <Input
                      id="description"
                      value={documentDescription}
                      onChange={(e) => setDocumentDescription(e.target.value)}
                    />
                  </div>
                  <Button type="submit">Upload Document</Button>
                </form>
              </CardContent>
            </Card>

            <Card>
              <CardHeader>
                <CardTitle>Recent Documents</CardTitle>
              </CardHeader>
              <CardContent>
                <Table>
                  <TableHeader>
                    <TableRow>
                      <TableHead>Title</TableHead>
                      <TableHead>Category</TableHead>
                      <TableHead>Date</TableHead>
                      <TableHead>Actions</TableHead>
                    </TableRow>
                  </TableHeader>
                  <TableBody>
                    {filteredDocuments.slice(0, 5).map((doc) => (
                      <TableRow key={doc.id}>
                        <TableCell>{doc.title}</TableCell>
                        <TableCell>{doc.category}</TableCell>
                        <TableCell>{doc.date}</TableCell>
                        <TableCell>
                          <Button variant="ghost" size="sm">View</Button>
                          <Button variant="ghost" size="sm">Download</Button>
                          <Button 
                            variant="ghost" 
                            size="sm"
                            onClick={() => handleDeleteDocument(doc.id)}
                          >
                            Delete
                          </Button>
                        </TableCell>
                      </TableRow>
                    ))}
                  </TableBody>
                </Table>
              </CardContent>
            </Card>
          </>
        )}

        {activeTab === 'users' && (
          <Card>
            <CardHeader>
              <CardTitle>User Management</CardTitle>
            </CardHeader>
            <CardContent>
              <form onSubmit={handleAddUser} className="space-y-4">
                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  <div className="space-y-2">
                    <Label htmlFor="new-email">Email</Label>
                    <Input
                      id="new-email"
                      type="email"
                      value={newUserEmail}
                      onChange={(e) => setNewUserEmail(e.target.value)}
                      required
                    />
                  </div>
                  <div className="space-y-2">
                    <Label htmlFor="new-password">Password</Label>
                    <Input
                      id="new-password"
                      type="password"
                      value={newUserPassword}
                      onChange={(e) => setNewUserPassword(e.target.value)}
                      required
                    />
                  </div>
                </div>
                <Button type="submit">Add User</Button>
              </form>

              <div className="mt-8">
                <h3 className="text-lg font-semibold mb-4">Existing Users</h3>
                <Table>
                  <TableHeader>
                    <TableRow>
                      <TableHead>Email</TableHead>
                      <TableHead>Actions</TableHead>
                    </TableRow>
                  </TableHeader>
                  <TableBody>
                    {users.map((user, index) => (
                      <TableRow key={index}>
                        <TableCell>{user.email}</TableCell>
                        <TableCell>
                          <Button variant="ghost" size="sm">Edit</Button>
                          <Button variant="ghost" size="sm">Delete</Button>
                        </TableCell>
                      </TableRow>
                    ))}
                  </TableBody>
                </Table>
              </div>
            </CardContent>
          </Card>
        )}

        {activeTab === 'notices' && (
          <Card>
            <CardHeader>
              <CardTitle>Notices</CardTitle>
            </CardHeader>
            <CardContent>
              <Table>
                <TableHeader>
                  <TableRow>
                    <TableHead>Title</TableHead>
                    <TableHead>Date</TableHead>
                    <TableHead>Actions</TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  {notices.map((notice) => (
                    <TableRow key={notice.id}>
                      <TableCell>{notice.title}</TableCell>
                      <TableCell>{notice.date}</TableCell>
                      <TableCell>
                        <Button variant="ghost" size="sm">View</Button>
                        <Button variant="ghost" size="sm">Edit</Button>
                      </TableCell>
                    </TableRow>
                  ))}
                </TableBody>
              </Table>
            </CardContent>
          </Card>
        )}

        {activeTab === 'settings' && (
          <Card>
            <CardHeader>
              <CardTitle>Settings</CardTitle>
            </CardHeader>
            <CardContent>
              <p>Settings content would go here.</p>
            </CardContent>
          </Card>
        )}
      </div>
    </div>
  )
}
