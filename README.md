# Calender-app
npx create-react-app calendar-app
cd calendar-app
npm install react-router-dom tailwindcss date-fns
/calendar-app
  /src
    /components
      /AdminModule
        - CompanyForm.js
        - CompanyList.js
      /UserModule
        - Dashboard.js
        - CommunicationActionModal.js
      /Shared
        - Header.js
        - Footer.js
    /contexts
      - CompanyContext.js
    - App.js
    - index.js
import React, { createContext, useContext, useState } from 'react';

// Create Context for managing company data
const CompanyContext = createContext();

// Custom hook to use Company Context
export const useCompany = () => {
  return useContext(CompanyContext);
};

// Provider Component for wrapping the App
export const CompanyProvider = ({ children }) => {
  const [companies, setCompanies] = useState([]);

  const addCompany = (company) => {
    setCompanies((prev) => [...prev, company]);
  };

  const editCompany = (updatedCompany) => {
    setCompanies((prev) =>
      prev.map((company) =>
        company.id === updatedCompany.id ? updatedCompany : company
      )
    );
  };

  const deleteCompany = (id) => {
    setCompanies((prev) => prev.filter((company) => company.id !== id));
  };

  return (
    <CompanyContext.Provider
      value={{ companies, addCompany, editCompany, deleteCompany }}
    >
      {children}
    </CompanyContext.Provider>
  );
};
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import { CompanyProvider } from './contexts/CompanyContext';
import Dashboard from './components/UserModule/Dashboard';
import CompanyList from './components/AdminModule/CompanyList';
import Header from './components/Shared/Header';
import Footer from './components/Shared/Footer';

function App() {
  return (
    <CompanyProvider>
      <Router>
        <Header />
        <Switch>
          <Route path="/admin" component={CompanyList} />
          <Route path="/" component={Dashboard} />
        </Switch>
        <Footer />
      </Router>
    </CompanyProvider>
  );
}

export default App;
import React from 'react';
import { useCompany } from '../../contexts/CompanyContext';
import { formatDistanceToNow } from 'date-fns';

const Dashboard = () => {
  const { companies } = useCompany();

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">Dashboard</h1>
      <table className="min-w-full border-collapse border border-gray-300">
        <thead>
          <tr>
            <th className="px-4 py-2">Company Name</th>
            <th className="px-4 py-2">Last Communication</th>
            <th className="px-4 py-2">Next Scheduled Communication</th>
          </tr>
        </thead>
        <tbody>
          {companies.map((company) => {
            const nextCommunicationDate = new Date(company.nextCommunication);
            const isOverdue = new Date() > nextCommunicationDate;

            return (
              <tr
                key={company.id}
                className={`${
                  isOverdue ? 'bg-red-100' : 'bg-green-100'
                } hover:bg-gray-200`}
              >
                <td className="px-4 py-2">{company.name}</td>
                <td className="px-4 py-2">
                  {formatDistanceToNow(new Date(company.lastCommunication))} ago
                </td>
                <td className="px-4 py-2">
                  {formatDistanceToNow(nextCommunicationDate)} away
                </td>
              </tr>
            );
          })}
        </tbody>
      </table>
    </div>
  );
};

export default Dashboard;
import React, { useState } from 'react';
import { useCompany } from '../../contexts/CompanyContext';

const CompanyForm = ({ company }) => {
  const { addCompany, editCompany } = useCompany();
  const [formData, setFormData] = useState({
    name: company?.name || '',
    lastCommunication: company?.lastCommunication || '',
    nextCommunication: company?.nextCommunication || '',
  });

  const handleSubmit = (e) => {
    e.preventDefault();

    const companyData = {
      ...formData,
      id: company?.id || Date.now(),
    };

    if (company) {
      editCompany(companyData);
    } else {
      addCompany(companyData);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="p-4 border shadow-md">
      <input
        type="text"
        name="name"
        placeholder="Company Name"
        value={formData.name}
        onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        className="border p-2 mb-4 w-full"
        required
      />
      <input
        type="datetime-local"
        name="lastCommunication"
        value={formData.lastCommunication}
        onChange={(e) =>
          setFormData({ ...formData, lastCommunication: e.target.value })
        }
        className="border p-2 mb-4 w-full"
        required
      />
      <input
        type="datetime-local"
        name="nextCommunication"
        value={formData.nextCommunication}
        onChange={(e) =>
          setFormData({ ...formData, nextCommunication: e.target.value })
        }
        className="border p-2 mb-4 w-full"
        required
      />
      <button
        type="submit"
        className="bg-blue-500 text-white p-2 rounded"
      >
        {company ? 'Edit Company' : 'Add Company'}
      </button>
    </form>
  );
};

export default CompanyForm;
import React from 'react';
import { useCompany } from '../../contexts/CompanyContext';
import CompanyForm from './CompanyForm';

const CompanyList = () => {
  const { companies, deleteCompany } = useCompany();

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">Company List</h1>
      <CompanyForm />
      <table className="min-w-full border-collapse border border-gray-300 mt-4">
        <thead>
          <tr>
            <th className="px-4 py-2">Company Name</th>
            <th className="px-4 py-2">Actions</th>
          </tr>
        </thead>
        <tbody>
          {companies.map((company) => (
            <tr key={company.id}>
              <td className="px-4 py-2">{company.name}</td>
              <td className="px-4 py-2">
                <button
                  onClick={() => deleteCompany(company.id)}
                  className="bg-red-500 text-white px-4 py-2 rounded"
                >
                  Delete
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default CompanyList;
const Header = () => {
  return (
    <header className="bg-gray-800 text-white p-4">
      <h1 className="text-xl font-bold">Calendar App</h1>
    </header>
  );
};

export default Header;
const Footer = () => {
  return (
    <footer className="bg-gray-800 text-white p-4 text-center">
      <p>&copy; 2025 Calendar App</p>
    </footer>
  );
};

export default Footer;
cd /path/to/your/calendar-app
git init
git remote add origin https://github.com/your-username/calendar-app.git
git add .
git commit -m "Initial commit for Calendar App"
git push -u origin master
git push -u origin main
