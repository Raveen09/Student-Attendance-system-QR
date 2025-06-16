<html>
<head>
    <title>Attendance System</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://unpkg.com/react/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.3/css/all.min.css"></link>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.3.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/0.4.1/html2canvas.min.js"></script>
    <style>
        .marquee {
            width: 100%;
            overflow: hidden;
            white-space: nowrap;
            box-sizing: border-box;
            animation: marquee 15s linear infinite;
        }
        @keyframes marquee {
            0% { transform: translate(100%, 0); }
            100% { transform: translate(-100%, 0); }
        }
        .background-image {
            background-image: url('https://gurugramuniversity.ac.in/img/logo.jpg');
            background-size: cover;
            background-position: center;
        }
        @media print {
            body * {
                visibility: hidden;
            }
        }
    </style>
</head>
<body class="bg-yellow-100 font-roboto">
    <div class="marquee bg-yellow-300 py-2 text-center text-lg font-bold">
        Gurugram University
    </div>
    <div id="root"></div>
    <script type="text/babel">
        const { useState, useEffect } = React;

        const App = () => {
            const [role, setRole] = useState(null);
            const [attendanceList, setAttendanceList] = useState([]);
            const [validQRCode, setValidQRCode] = useState(null);
            const [captcha, setCaptcha] = useState('');
            const [captchaInput, setCaptchaInput] = useState('');
            const [captchaVerified, setCaptchaVerified] = useState(false);
            const [lastAttendanceTime, setLastAttendanceTime] = useState(null);

            useEffect(() => {
                generateCaptcha();
                loadAttendanceList();
            }, []);

            const generateCaptcha = () => {
                const chars = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
                let captchaText = '';
                for (let i = 0; i < 6; i++) {
                    captchaText += chars[Math.floor(Math.random() * chars.length)];
                }
                setCaptcha(captchaText);
            };

            const verifyCaptcha = () => {
                if (captcha === captchaInput) {
                    setCaptchaVerified(true);
                } else {
                    alert('Invalid captcha. Please try again.');
                    generateCaptcha();
                    setCaptchaInput('');
                }
            };

            const loadAttendanceList = () => {
                const storedList = JSON.parse(localStorage.getItem('attendanceList')) || [];
                const now = new Date();
                const filteredList = storedList.filter(item => (now - new Date(item.timestamp)) < 7 * 24 * 60 * 60 * 1000);
                setAttendanceList(filteredList);
            };

            const saveAttendanceList = (list) => {
                localStorage.setItem('attendanceList', JSON.stringify(list));
            };

            return (
                <div className="min-h-screen flex flex-col items-center justify-center p-4 background-image">
                    {!captchaVerified && (
                        <div className="space-y-4 text-center bg-yellow-100 bg-opacity-75 p-4 rounded">
                            <h1 className="text-3xl font-bold">Enter Captcha to Proceed</h1>
                            <div className="mt-4">
                                <p className="text-lg">Captcha: <strong>{captcha}</strong></p>
                                <input type="text" placeholder="Enter Captcha" value={captchaInput} onChange={(e) => setCaptchaInput(e.target.value)} className="px-4 py-2 border rounded w-full" />
                                <button onClick={verifyCaptcha} className="px-4 py-2 bg-blue-500 text-white rounded w-full mt-2">Verify Captcha</button>
                            </div>
                        </div>
                    )}
                    {captchaVerified && !role && (
                        <div className="space-y-4 text-center bg-yellow-100 bg-opacity-75 p-4 rounded">
                            <h1 className="text-3xl font-bold">Choose Your Role</h1>
                            <button onClick={() => setRole('student')} className="px-4 py-2 bg-blue-500 text-white rounded w-full sm:w-auto">Student</button>
                            <button onClick={() => setRole('teacher')} className="px-4 py-2 bg-green-500 text-white rounded w-full sm:w-auto">Teacher</button>
                        </div>
                    )}
                    {role === 'student' && <StudentSection goBack={() => setRole(null)} setAttendanceList={setAttendanceList} validQRCode={validQRCode} lastAttendanceTime={lastAttendanceTime} setLastAttendanceTime={setLastAttendanceTime} saveAttendanceList={saveAttendanceList} />}
                    {role === 'teacher' && <TeacherSection goBack={() => setRole(null)} attendanceList={attendanceList} setValidQRCode={setValidQRCode} saveAttendanceList={saveAttendanceList} />}
                </div>
            );
        };

        const StudentSection = ({ goBack, setAttendanceList, validQRCode, lastAttendanceTime, setLastAttendanceTime, saveAttendanceList }) => {
            const [name, setName] = useState('');
            const [course, setCourse] = useState('');
            const [rollNo, setRollNo] = useState('');
            const [qrCode, setQrCode] = useState('');
            const [className, setClassName] = useState('');

            const handleScan = () => {
                const now = new Date();
                if (lastAttendanceTime && (now - lastAttendanceTime) < 25 * 60 * 1000) {
                    alert('You can only submit attendance once every 25 minutes.');
                    return;
                }

                if (qrCode === validQRCode) {
                    alert('Attendance marked successfully!');
                    const newAttendance = { name, course, rollNo, qrCode, className, timestamp: now };
                    const updatedList = [...attendanceList, newAttendance];
                    setAttendanceList(updatedList);
                    saveAttendanceList(updatedList);
                    setLastAttendanceTime(now);
                } else {
                    alert('Invalid QR code!');
                }
            };

            const handleSubmit = () => {
                const now = new Date();
                if (lastAttendanceTime && (now - lastAttendanceTime) < 25 * 60 * 1000) {
                    alert('You can only submit attendance once every 25 minutes.');
                    return;
                }

                if (qrCode === validQRCode) {
                    alert('Attendance submitted successfully!');
                    const newAttendance = { name, course, rollNo, qrCode, className, timestamp: now };
                    const updatedList = [...attendanceList, newAttendance];
                    setAttendanceList(updatedList);
                    saveAttendanceList(updatedList);
                    setLastAttendanceTime(now);
                    goBack();
                } else {
                    alert('Invalid QR code!');
                }
            };

            return (
                <div className="space-y-4 w-full max-w-md bg-yellow-100 bg-opacity-75 p-4 rounded">
                    <h2 className="text-2xl font-bold">Student Section</h2>
                    <button onClick={goBack} className="px-4 py-2 bg-gray-500 text-white rounded w-full">Go Back</button>
                    <input type="text" placeholder="Enter Class Name" value={className} onChange={(e) => setClassName(e.target.value)} className="px-4 py-2 border rounded w-full" />
                    <input type="text" placeholder="Name" value={name} onChange={(e) => setName(e.target.value)} className="px-4 py-2 border rounded w-full" />
                    <input type="text" placeholder="Course" value={course} onChange={(e) => setCourse(e.target.value)} className="px-4 py-2 border rounded w-full" />
                    <input type="text" placeholder="Roll No." value={rollNo} onChange={(e) => setRollNo(e.target.value)} className="px-4 py-2 border rounded w-full" />
                    <input type="text" placeholder="Enter QR Code" value={qrCode} onChange={(e) => setQrCode(e.target.value)} className="px-4 py-2 border rounded w-full" />
                    <button onClick={handleScan} className="px-4 py-2 bg-blue-500 text-white rounded w-full">Scan QR Code</button>
                    <button onClick={handleSubmit} className="px-4 py-2 bg-green-500 text-white rounded w-full">Submit Attendance</button>
                </div>
            );
        };

        const TeacherSection = ({ goBack, attendanceList, setValidQRCode, saveAttendanceList }) => {
            const [qrCode, setQrCode] = useState('');

            const generateQRCode = () => {
                // Generate a new QR code
                const newQRCode = `QR_${Math.random().toString(36).substr(2, 9)}`;
                setQrCode(newQRCode);
                setValidQRCode(newQRCode);
                alert('QR Code generated successfully!');

                // Invalidate the QR code after 5 minutes
                setTimeout(() => {
                    setValidQRCode(null);
                    alert('QR Code has expired.');
                }, 5 * 60 * 1000);
            };

            const downloadAttendanceList = () => {
                const fileData = JSON.stringify(attendanceList, null, 2);
                const blob = new Blob([fileData], { type: 'application/json' });
                const url = URL.createObjectURL(blob);

                if (blob.size > 1024 * 1024) { // If file size is greater than 1MB, download as PDF
                    const { jsPDF } = window.jspdf;
                    const doc = new jsPDF();
                    doc.text(fileData, 10, 10);
                    doc.save('attendance_list.pdf');
                } else { // Otherwise, download as image
                    html2canvas(document.querySelector("#attendanceList")).then(canvas => {
                        const imgData = canvas.toDataURL('image/png');
                        const a = document.createElement('a');
                        a.href = imgData;
                        a.download = 'attendance_list.png';
                        a.click();
                    });
                }

                URL.revokeObjectURL(url);
            };

            return (
                <div className="min-h-screen flex flex-col items-center justify-center p-4 background-image">
                    <div className="space-y-4 w-full max-w-md bg-yellow-100 bg-opacity-75 p-4 rounded">
                        <h2 className="text-2xl font-bold">Teacher Section</h2>
                        <button onClick={goBack} className="px-4 py-2 bg-gray-500 text-white rounded w-full">Go Back</button>
                        <button onClick={generateQRCode} className="px-4 py-2 bg-green-500 text-white rounded w-full">Generate QR Code</button>
                        {qrCode && (
                            <div className="mt-4">
                                <p className="text-lg">Generated QR Code:</p>
                                <div className="p-4 border rounded bg-gray-200">{qrCode}</div>
                            </div>
                        )}
                        <div className="mt-4" id="attendanceList">
                            <h3 className="text-xl font-bold">Attendance List</h3>
                            <ul className="list-disc pl-5">
                                {attendanceList.map((student, index) => (
                                    <li key={index} className="mt-2">
                                        <p><strong>Name:</strong> {student.name}</p>
                                        <p><strong>Course:</strong> {student.course}</p>
                                        <p><strong>Roll No.:</strong> {student.rollNo}</p>
                                        <p><strong>QR Code:</strong> {student.qrCode}</p>
                                        <p><strong>Class Name:</strong> {student.className}</p>
                                        <p><strong>Timestamp:</strong> {new Date(student.timestamp).toLocaleString()}</p>
                                    </li>
                                ))}
                            </ul>
                            <button onClick={downloadAttendanceList} className="px-4 py-2 bg-blue-500 text-white rounded w-full mt-4">Download Attendance List</button>
                        </div>
                    </div>
                </div>
            );
        };

        ReactDOM.render(<App />, document.getElementById('root'));
    </script>
</body>
</html>C
