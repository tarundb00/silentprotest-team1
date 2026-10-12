# silentprotest-team1
asdfghjkl
import React, { useState, useRef, useEffect } from 'react';
import {
  FaUser, FaEnvelope, FaPhone, FaCalendarAlt,
  FaVenusMars, FaMapMarkerAlt, FaCamera, FaLock, FaEye, FaEyeSlash,
  FaChalkboardTeacher, FaEdit, FaTrash, FaPlus, FaTimes,
} from 'react-icons/fa';
import AdminLayout from './AdminLayout';
import './AddTeacher.css';
import useToast from '../components/useToast';
import Toast from '../components/Toast';
import { fetchTeachers, createTeacher, updateTeacher, deleteTeacher, uploadProfilePicture } from '../services/api';

const initialState = {
  firstName: '', lastName: '', email: '', phone: '',
  dateOfBirth: '', gender: '', address: '', profilePicture: null,
  password: '', confirmPassword: '',
};

export default function AddTeacher() {
  const [form, setForm] = useState(initialState);
  const [preview, setPreview] = useState(null);
  const [teachers, setTeachers] = useState([]);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [editingId, setEditingId] = useState(null);
  const [deleteTargetId, setDeleteTargetId] = useState(null);
  const [showPass, setShowPass]        = useState(false);
  const [showConfirm, setShowConfirm]  = useState(false);
  const [passError, setPassError]      = useState('');
  const fileRef = useRef(null);
  const { toasts, removeToast, toast } = useToast();

  useEffect(() => {
    const loadTeachers = async () => {
      try {
        const data = await fetchTeachers();
        setTeachers(Array.isArray(data) ? data : []);
      } catch (error) {
        console.error('Failed to load teachers', error);
        toast.error(error.message || 'Unable to load teachers');
      }
    };
    loadTeachers();
  }, []);

  const resetForm = () => { setForm({ ...initialState }); setPreview(null); setEditingId(null); };

  const openModal = (teacher = null) => {
    if (teacher) {
      setForm({ ...teacher });
      setPreview(teacher.profilePictureUrl || null);
      setEditingId(teacher.id);
    } else {
      resetForm();
    }
    setIsModalOpen(true);
  };

  const closeModal = () => { setIsModalOpen(false); resetForm(); };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm((p) => ({ ...p, [name]: value }));
  };

  const handlePhoto = async (e) => {
    const file = e.target.files[0];
    if (!file) return;
    // Show local preview immediately
    const localUrl = URL.createObjectURL(file);
    setPreview(localUrl);
    try {
      const data = await uploadProfilePicture(file);
      if (data.url) {
        const backendUrl = `http://localhost:8080${data.url}`;
        setForm((p) => ({ ...p, profilePictureUrl: backendUrl }));
        setPreview(backendUrl);
      }
    } catch {
      // keep local preview if upload fails
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setPassError('');
    if (!editingId && !form.password) {
      setPassError('Password is required'); return;
    }
    if (form.password && form.password !== form.confirmPassword) {
      setPassError('Passwords do not match'); return;
    }
    if (form.password && form.password.length < 6) {
      setPassError('Password must be at least 6 characters'); return;
    }
    const { confirmPassword, ...rest } = form;
    const payload = { ...rest, profilePictureUrl: form.profilePictureUrl || null };
    if (!payload.password) delete payload.password;
    try {
      if (editingId) {
        const updated = await updateTeacher(editingId, payload);
        setTeachers((prev) => prev.map((t) => (t.id === editingId ? updated : t)));
        toast.success('Teacher updated successfully');
      } else {
        const created = await createTeacher(payload);
        setTeachers((prev) => [created, ...prev]);
        toast.success('Teacher added successfully');
      }
      closeModal();
    } catch (error) {
      console.error('Failed to save teacher', error);
      toast.error(error.message || 'Unable to save teacher');
    }
  };

  const handleDelete = async () => {
    if (!deleteTargetId) return;
    try {
      await deleteTeacher(deleteTargetId);
      setTeachers((prev) => prev.filter((t) => t.id !== deleteTargetId));
      setDeleteTargetId(null);
      toast.success('Teacher deleted successfully');
    } catch (error) {
      console.error('Failed to delete teacher', error);
      toast.error(error.message || 'Unable to delete teacher');
    }
  };

  const getInitials = (t) => `${t.firstName?.[0] || ''}${t.lastName?.[0] || ''}`.toUpperCase();

  return (
    <>
    <AdminLayout title="Teacher Management" subtitle="Manage and view all registered teachers">


    </AdminLayout>
    <Toast toasts={toasts} removeToast={removeToast} />
    </>
  );
}
