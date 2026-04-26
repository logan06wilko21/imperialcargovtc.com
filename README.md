# imperialcargovtc.com
import { useState, useEffect } from "react";

export default function ImperialCargoVTC() {
  const [user, setUser] = useState(null);
  const [form, setForm] = useState({ username: "", password: "" });

  const [jobForm, setJobForm] = useState({ miles: "", cargo: "", route: "" });
  const [jobs, setJobs] = useState([]);

  const [appForm, setAppForm] = useState({ discord: "", experience: "", reason: "" });
  const [applications, setApplications] = useState([]);
  const [approvedDrivers, setApprovedDrivers] = useState([]);

  useEffect(() => {
    const savedJobs = localStorage.getItem("ic_jobs");
    if (savedJobs) setJobs(JSON.parse(savedJobs));

    const savedApps = localStorage.getItem("ic_apps");
    if (savedApps) setApplications(JSON.parse(savedApps));

    const savedApproved = localStorage.getItem("ic_approved");
    if (savedApproved) setApprovedDrivers(JSON.parse(savedApproved));
  }, []);

  const login = () => {
    if (!form.username || !form.password) return;

    const isAdmin = form.username.toLowerCase().includes("admin");
    const isApproved = approvedDrivers.includes(form.username);

    if (!isAdmin && !isApproved) {
      alert("You are not approved as a driver yet.");
      return;
    }

    setUser({
      username: form.username,
      role: isAdmin ? "admin" : "driver",
      rank: isAdmin ? "Staff Manager" : "Junior Driver",
      miles: 1240,
      deliveries: 32
    });
  };

  const logout = () => setUser(null);

  const submitJob = () => {
    if (!jobForm.miles || !jobForm.cargo || !jobForm.route) return;

    const newJob = {
      id: Date.now(),
      driver: user.username,
      miles: parseInt(jobForm.miles),
      cargo: jobForm.cargo,
      route: jobForm.route,
      date: new Date().toLocaleString()
    };

    const updatedJobs = [newJob, ...jobs];
    setJobs(updatedJobs);
    localStorage.setItem("ic_jobs", JSON.stringify(updatedJobs));

    setUser({
      ...user,
      miles: user.miles + newJob.miles,
      deliveries: user.deliveries + 1
    });

    setJobForm({ miles: "", cargo: "", route: "" });
  };

  const submitApplication = () => {
    if (!appForm.discord || !appForm.experience || !appForm.reason) return;

    const newApp = {
      id: Date.now(),
      discord: appForm.discord,
      experience: appForm.experience,
      reason: appForm.reason,
      status: "pending",
      date: new Date().toLocaleString()
    };

    const updated = [newApp, ...applications];
    setApplications(updated);
    localStorage.setItem("ic_apps", JSON.stringify(updated));

    setAppForm({ discord: "", experience: "", reason: "" });
  };

  const approveApp = (app) => {
    const updatedApps = applications.map(a =>
      a.id === app.id ? { ...a, status: "approved" } : a
    );

    const updatedApproved = [...approvedDrivers, app.discord];

    setApplications(updatedApps);
    setApprovedDrivers(updatedApproved);

    localStorage.setItem("ic_apps", JSON.stringify(updatedApps));
    localStorage.setItem("ic_approved", JSON.stringify(updatedApproved));
  };

  const rejectApp = (app) => {
    const updatedApps = applications.map(a =>
      a.id === app.id ? { ...a, status: "rejected" } : a
    );

    setApplications(updatedApps);
    localStorage.setItem("ic_apps", JSON.stringify(updatedApps));
  };

  const deleteJob = (id) => {
    const updated = jobs.filter(j => j.id !== id);
    setJobs(updated);
    localStorage.setItem("ic_jobs", JSON.stringify(updated));
  };

  // ADMIN PANEL
  if (user && user.role === "admin") {
    return (
      <div style={styles.page}>
        <h1>🛠️ Imperial Cargo Admin Panel</h1>

        <button onClick={logout} style={styles.logout}>Logout</button>

        <div style={styles.grid}>
          <div style={styles.card}>📦 Jobs: {jobs.length}</div>
          <div style={styles.card}>👥 Drivers: {approvedDrivers.length}</div>
          <div style={styles.card}>📝 Applications: {applications.length}</div>
        </div>

        <h2>Driver Applications</h2>
        {applications.map(app => (
          <div key={app.id} style={styles.item}>
            <b>{app.discord}</b>
            <p>{app.experience}</p>
            <p>{app.reason}</p>
            <p>Status: {app.status}</p>
            <button onClick={() => approveApp(app)}>Approve</button>
            <button onClick={() => rejectApp(app)}>Reject</button>
          </div>
        ))}

        <h2>Job Logs</h2>
        {jobs.map(job => (
          <div key={job.id} style={styles.item}>
            <b>{job.cargo}</b> - {job.miles} miles
            <p>{job.driver}</p>
            <button onClick={() => deleteJob(job.id)}>Delete</button>
          </div>
        ))}
      </div>
    );
  }

  // DRIVER DASHBOARD
  if (user) {
    return (
      <div style={styles.page}>
        <h1>🚛 Driver Dashboard</h1>
        <button onClick={logout} style={styles.logout}>Logout</button>

        <div style={styles.grid}>
          <div style={styles.card}>👤 {user.username}</div>
          <div style={styles.card}>⭐ {user.rank}</div>
          <div style={styles.card}>🛣️ {user.miles} miles</div>
        </div>

        <h2>Submit Job</h2>
        <input placeholder="Miles" onChange={e => setJobForm({ ...jobForm, miles: e.target.value })} />
        <input placeholder="Cargo" onChange={e => setJobForm({ ...jobForm, cargo: e.target.value })} />
        <input placeholder="Route" onChange={e => setJobForm({ ...jobForm, route: e.target.value })} />
        <button onClick={submitJob}>Submit</button>
      </div>
    );
  }

  // RECRUITMENT PAGE
  return (
    <div style={styles.page}>
      <h1>🚛 Imperial Cargo Recruitment</h1>

      <h2>Apply</h2>
      <input placeholder="Discord" onChange={e => setAppForm({ ...appForm, discord: e.target.value })} />
      <input placeholder="Experience" onChange={e => setAppForm({ ...appForm, experience: e.target.value })} />
      <textarea placeholder="Why join?" onChange={e => setAppForm({ ...appForm, reason: e.target.value })} />
      <button onClick={submitApplication}>Submit Application</button>

      <h2>Login</h2>
      <input placeholder="Username" onChange={e => setForm({ ...form, username: e.target.value })} />
      <input type="password" placeholder="Password" onChange={e => setForm({ ...form, password: e.target.value })} />
      <button onClick={login}>Login</button>
    </div>
  );
}

const styles = {
  page: { padding: 20, fontFamily: "Arial", background: "#0B1F3B", color: "white", minHeight: "100vh" },
  grid: { display: "flex", gap: 10, marginBottom: 20 },
  card: { background: "rgba(255,255,255,0.1)", padding: 10, borderRadius: 8 },
  item: { background: "rgba(255,255,255,0.1)", padding: 10, marginBottom: 10, borderRadius: 8 },
  logout: { marginBottom: 10 }
};
